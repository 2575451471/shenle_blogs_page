---
publish: true
tags:
  - 代码小记
---
# 贪吃蛇代码  
## 硬件连接

| 硬件功能 | 引脚 | 说明 |
| :--- | :--- | :--- |
| 点阵列端口 | P0 | 8位列选，低电平有效（共阴） |
| 74HC595 | P3^6 (SH_CP) | 移位时钟 |
| 74HC595 | P3^5 (ST_CP) | 存储时钟（锁存） |
| 74HC595 | P3^4 (DS) | 串行数据 |
| 方向按键 | P2^7 | 上，按键按下为低电平 |
| 方向按键 | P2^6 | 下，按键按下为低电平 |
| 方向按键 | P2^5 | 左，按键按下为低电平 |
| 方向按键 | P2^4 | 右，按键按下为低电平 |
| 外部中断0 | P3^2 (INT0) | 用于重启游戏，下降沿触发 |

## 定义
1. 
```c
#define COL_PORT P0          // 点阵列端口（共阴，低电平有效）
```
>P0连接点阵的列线，共阴极，低电平时点亮对应的列。
- - -  
2. 
```c
sbit SH_CP = P3^6;           // 移位时钟
sbit ST_CP = P3^5;          // 存储时钟
sbit DS    = P3^4;          // 串行数据
```
>三个引脚控制74HC595，用于向点阵的行线发送数据。74HC595是一个串入并出移位寄存器，这里用来扩展IO口，控制8行。
- - -
3. 
```c
sbit KEY_UP    = P2^7;      // 上
sbit KEY_DOWN  = P2^6;      // 下
sbit KEY_LEFT  = P2^5;      // 左
sbit KEY_RIGHT = P2^4;      // 右
```
>四个独立按键接在P2口的高四位，低电平有效。  
- - -
## 数据结构、游戏配置
1. 
```c
#define MAX_SNAKE_LENGTH    40
#define INIT_SNAKE_LENGTH   5
```
>蛇的最大长度和初始长度。  
- - - 
2. 
```c
struct Snake {
    unsigned char x[MAX_SNAKE_LENGTH];
    unsigned char y[MAX_SNAKE_LENGTH];
    unsigned char length;
    char dir_x;
    char dir_y;
} snake;
```
>蛇的结构体：   
>x和y数组存储每一节身体的坐标，  
>length为当前长度，  
>dir_x和dir_y为当前移动方向（取值为-1、0、1）。  
- - -
3. 
```c
struct Food {
    unsigned char x;
    unsigned char y;
    unsigned char exist;
} food;
```
>食物的结构体：坐标和存在标志。
- - -
4. 
```c
unsigned char game_speed = 45;
unsigned char speed_counter = 0;
bit wall_collision = 0;
unsigned char random_seed = 0;
unsigned char food_counter = 0;
unsigned char score = 0;

bit game_started = 0;           // 游戏启动标志
bit game_over = 0;             // 游戏结束标志
bit animation_complete = 0;    // 动画完成（全亮）标志
unsigned char lit_rows_mask = 0; // 累积点亮的行掩码（位0~7，0为最下行）
bit restart_flag = 0;          // 外部中断请求重启标志
```
>game_speed控制移动速度（循环次数），  
>speed_counter用于计时；  
>wall_collision记录是否撞墙；  
>random_seed和food_counter用于随机数生成；  
>score记录得分（未显示）；  
>几个标志位控制游戏状态和动画。 
- - -

## 74HC595驱动
```c
void hc595_send_byte(unsigned char dat) {
    unsigned char i;
    SH_CP = 0;
    ST_CP = 0;
    for(i = 0; i < 8; i++) {
        DS = (dat & (0x80 >> i)) ? 1 : 0;
        SH_CP = 1;
        SH_CP = 0;
    }
    ST_CP = 1;
    ST_CP = 0;
}
```
>通过串行方式向595发送一个字节：  
>循环8次，每次在SH_CP上升沿将数据位移入寄存器，最后ST_CP上升沿将数据锁存到输出引脚。  
>这里发送的是行数据，对应点阵的8行。  
- - -
## 定时器0 
```c
void timer0_init(void) {
    TMOD &= 0xF0;
    TMOD |= 0x01;      // 模式1：16位定时器
    TH0 = 0;
    TL0 = 0;
    TR0 = 1;           // 启动定时器0
}
```
>定时器0初始化为16位自由运行模式，从0开始计数，一直加到65535后溢出归零，循环不止。  
>不开启中断，只利用它的计数值作为随机数熵源。  
>每当按键被确认时，我们会读取当前TH0和TL0的值，混入随机种子，这样按键的随机时刻就带来了硬件随机性。  
- - -

## 蛇和食物的初始化
```c
void snake_init(void) {
    unsigned char i;
    snake.length = INIT_SNAKE_LENGTH;
    snake.dir_x = 1;
    snake.dir_y = 0;
    for(i = 0; i < snake.length; i++) {
        snake.x[i] = 4 - i;
        snake.y[i] = 4;
    }
}
```
>初始蛇为5节，水平向左，从(4,4)开始向右排列，即蛇头在(4,4)，身体依次在(3,4)、(2,4)、(1,4)、(0,4)。  
>这样蛇一开始就在屏幕中间偏右。  
```c
void food_init(void) {
    food.exist = 0;
}
void generate_food(void) {
    unsigned char i;
    bit valid_position;
    food_counter++;
    random_seed += food_counter;
    do {
        valid_position = 1;
        food.x = get_random() % 8;
        food.y = get_random() % 8;
        for(i = 0; i < snake.length; i++) {
            if(snake.x[i] == food.x && snake.y[i] == food.y) {
                valid_position = 0;
                break;
            }
        }
    } while(!valid_position);
    food.exist = 1;
}
```
>生成新食物时，先增加food_counter并累加到随机种子，然后循环生成随机坐标，直到不与蛇的任何一节重合。  
>这里用到了get_random()，但因为种子被不断扰动，所以食物位置具有一定的随机性。  
- - -
## 按键扫描、方向控制  
```c
unsigned char key_scan(void) {
    static unsigned char key_state = 0;
    static unsigned char key_debounce = 0;
    unsigned char key_val = 0;
    unsigned int timer_val;          // 声明局部变量
    
    switch(key_state) {
        case 0: // 检测是否有键按下
            if(KEY_UP == 0 || KEY_DOWN == 0 || KEY_LEFT == 0 || KEY_RIGHT == 0) {
                key_debounce = 0;
                key_state = 1;
            }
            break;
        case 1: // 消抖
            key_debounce++;
            if(key_debounce >= 10) {
                if(KEY_UP == 0)      key_val = 1;
                else if(KEY_DOWN == 0) key_val = 2;
                else if(KEY_LEFT == 0) key_val = 3;
                else if(KEY_RIGHT == 0) key_val = 4;
                else {
                    key_state = 0;
                    break;
                }
                if(key_val != 0) {
                    key_state = 2;
                    
                    // 按键确认时捕获定时器0计数值
                    timer_val = (TH0 << 8) | TL0;
                    random_seed += timer_val;
                    food_counter += (unsigned char)timer_val;
                    
                    return key_val;
                }
            }
            break;
        case 2: // 等待按键释放
            if(KEY_UP == 1 && KEY_DOWN == 1 && KEY_LEFT == 1 && KEY_RIGHT == 1) {
                key_state = 0;
            }
            break;
    }
    return 0;
}
```
>按键扫描采用状态机，实现消抖。  
>在确认按键有效后，读取定时器0当前的16位计数值，分别加到random_seed和food_counter上。这样每次按键的时间差异都会影响随机数，使得游戏不可预测。  
### 按键扫描状态机介绍
状态机通过 switch(key_state) 实现，分为空闲检测、消抖确认、等待释放三个状态，流程是：0 → 1 → 2 → 0（仅当有有效按键时）。  
__状态 0：空闲检测__  
1. 初始状态就是 0，函数每次调用都会先进入这个状态检查。  
2. 如果检测到 UP/DOWN/LEFT/RIGHT 任意一个按键引脚为 0（有按下迹象），立即：  
    - 重置消抖计数器 `key_debounce = 0`；
    - 将状态切换到 1（进入消抖阶段）。  
3. 如果无按键按下，保持状态 0，返回 0（无按键）。  

__状态 1：消抖确认（case 1）__   
1. 每次进入该状态，消抖计数器 `key_debounce` 加 1（累计消抖次数，对应硬件消抖时间，比如每次调用函数间隔 1ms，累计 10 次就是 10ms 消抖）。
2. 当计数器≥10（消抖时间达标），再次检测按键引脚：  
    - 如果是 UP 按下 → `key_val=1`；
    - 如果是 DOWN 按下 → `key_val=2`；
    - 如果是 LEFT 按下 → `key_val=3`；
    - 如果是 RIGHT 按下 → `key_val=4`；
    - 如果此时所有按键都不是 0（抖动导致的误触发），则切回状态 0，结束本次检测。  
3. 若确认有有效按键（`key_val≠0`）：
    - 切换状态到 2（等待释放）。
    - 捕获定时器 0 的计数值（`TH0` 高 8 位 + `TL0` 低 8 位），更新 `random_seed` 和 `food_counter`；
4. 如果计数器 < 10，不做其他操作，返回 0（消抖中）。    

__状态 2：等待释放（case 2）__  
1. 检测所有按键引脚是否都为 1（全部释放）。
2. 若全部释放，切回状态 0（回到空闲，准备下一次检测）；若未释放，保持状态 2，返回 0。
3. 这个状态的核心是：单次按键按下仅返回一次 `key_val`，直到按键松开后才允许检测下一次按键。

### 方向控制
- - -
```c
void update_direction(unsigned char key_val) {
    switch(key_val) {
        case 1: if(snake.dir_y == 0) { snake.dir_x = 0; snake.dir_y = -1; } break;
        case 2: if(snake.dir_y == 0) { snake.dir_x = 0; snake.dir_y = 1;  } break;
        case 3: if(snake.dir_x == 0) { snake.dir_x = -1; snake.dir_y = 0; } break;
        case 4: if(snake.dir_x == 0) { snake.dir_x = 1;  snake.dir_y = 0; } break;
    }
}
void set_start_direction(unsigned char key_val) {
    switch(key_val) {
        case 1: snake.dir_x = 0; snake.dir_y = -1; break;
        case 2: snake.dir_x = 0; snake.dir_y = 1;  break;
        case 3: snake.dir_x = -1; snake.dir_y = 0; break;
        case 4: snake.dir_x = 1;  snake.dir_y = 0; break;
    }
}
```
>方向控制函数：update_direction在游戏运行时调用，会检查当前方向，防止直接掉头。  
>set_start_direction用于游戏开始前的第一次按键，直接设置方向。  


## 移动逻辑、碰撞检测
```c
bit check_eat_food(void) {
    if(food.exist == 1 && snake.x[0] == food.x && snake.y[0] == food.y)
        return 1;
    return 0;
}
```
>检查蛇头是否吃到食物。  
- - -
```c
void move_snake(void) {
    char i;
    char new_x, new_y;
    new_x = snake.x[0] + snake.dir_x;
    new_y = snake.y[0] + snake.dir_y;
    if(new_x < 0 || new_x >= 8 || new_y < 0 || new_y >= 8) {
        wall_collision = 1;
        return;
    }
    if(check_eat_food()) {
        if(snake.length < MAX_SNAKE_LENGTH) {
            for(i = snake.length; i > 0; i--) {
                snake.x[i] = snake.x[i-1];
                snake.y[i] = snake.y[i-1];
            }
            snake.length++;
            score++;
            generate_food();
        }
    } else {
        for(i = snake.length - 1; i > 0; i--) {
            snake.x[i] = snake.x[i-1];
            snake.y[i] = snake.y[i-1];
        }
    }
    snake.x[0] = new_x;
    snake.y[0] = new_y;
}
```
___移动蛇的核心函数：___    
1. __计算新头部坐标__  
- snake.x[0]/snake.y[0] 是蛇头的当前坐标（数组下标 0 固定为头部）；
- snake.dir_x/snake.dir_y 是移动方向（由按键控制）：
    - 向右：dir_x=1, dir_y=0 → 新头部 x+1，y 不变；
    - 向左：dir_x=-1, dir_y=0 → 新头部 x-1，y 不变；
    - 向上：dir_x=0, dir_y=-1 → 新头部 y-1，x 不变；
    - 向下：dir_x=0, dir_y=1 → 新头部 y+1，x 不变；
2. __边界碰撞检测__
- 点阵是 8x8（坐标范围 0~7），若新头部 x/y <0 或 ≥8，说明撞墙：
    - 标记 wall_collision = 1（碰撞标志）；
    - 直接 return，不执行后续移动逻辑。
3. __分场景处理身体移动__    

| 情况 | 循环逻辑 | 操作描述 | 效果 |
| --- | --- | --- | --- |
| 未吃食物 | `i = length-1; i>0; i--` | 从尾部向前复制，尾部丢弃 | 长度不变，整体前进 |
| 吃食物 | `i = length; i>0; i--` | 从新增位置向前复制，尾部保留 | 长度 + 1，整体前进 |
- - -

## 点阵显示、游戏结束动画
```c
void matrix_display(void) {
    unsigned char i, j;
    unsigned char row_data;
    for(i = 0; i < 8; i++) {
        hc595_send_byte(0x00);
        COL_PORT = ~(0x01 << i);
        row_data = 0x00;

        // 正常显示蛇身和食物
        for(j = 0; j < snake.length; j++) {
            if(snake.x[j] == i) {
                row_data |= (0x01 << snake.y[j]);
            }
        }
        if(food.exist == 1 && food.x == i) {
            row_data |= (0x01 << food.y);
        }

        // 游戏结束动画：累积点亮行
        if(game_over) {
            if(!animation_complete) {
                row_data |= lit_rows_mask;
            } else {
                row_data = 0xFF;
            }
        }

        hc595_send_byte(row_data);
        delay_us(50);
    }
}
```
显示函数采用逐列扫描（i为列号，从0到7）。先发送行数据（595控制行），然后选通对应列（COL_PORT输出低电平）。正常显示时，根据蛇的坐标和食物坐标设置行数据（某列上哪些行应该点亮）。如果游戏结束且动画未完成，则将已累积点亮的行掩码（lit_rows_mask）加到当前列的行数据上，实现逐行累积点亮的效果；如果动画完成，则全列所有行都点亮（0xFF）。  

```c
void game_over_animation(void) {
    unsigned char row;
    unsigned int k;
    
    lit_rows_mask = 0x00;
    animation_complete = 0;
    
    for(row = 0; row < 8; row++) {
        lit_rows_mask |= (0x01 << row);   // 点亮当前行，并保持之前点亮的所有行
        for(k = 0; k < 20; k++) {
            matrix_display();
        }
    }
    
    animation_complete = 1;
    lit_rows_mask = 0xFF;
    for(k = 0; k < 100; k++) {
        matrix_display();
    }
}
```
游戏结束动画：从最下面一行（row=0）开始，每次点亮一行，并且保持之前点亮的所有行，这样屏幕会从下往上逐渐全部点亮。每步循环20次matrix_display()，约20ms（取决于延时），总共约160ms。最后全亮约100ms。注意这里matrix_display()内部会根据game_over和lit_rows_mask来显示，所以动画过程中蛇身和食物仍然可见，但会被累积点亮的行覆盖，最终全亮。  

## 外部中断  
```c
void int0_isr(void) interrupt 0 {
    restart_flag = 1;           // 请求重启
    EX0 = 0;                   // 立即关闭外部中断，防止重复触发
}
```
外部中断0下降沿触发。当游戏结束后，主循环会开启外部中断，等待用户按下外部按钮（接在INT0引脚）。按下后，中断服务程序设置restart_flag，并立即关闭中断，防止多次触发。主循环检测到restart_flag后会重新初始化游戏。  



## main主流程  
```c
void main(void) {
    unsigned char key_val;

    // 外部中断0初始化：下降沿触发，全局中断允许，初始关闭
    IT0 = 1;    // 下降沿触发
    EA  = 1;    // 开总中断
    EX0 = 0;    // 禁止外部中断0，游戏结束后才使能

    game_init();
    timer0_init();             // 启动定时器0，作为硬件随机熵源

    while(1) {
        key_val = key_scan();

        // ----- 按键处理（仅在游戏进行时响应方向，未开始时启动）-----
        if(key_val != 0 && !game_over) {   // 游戏结束时不处理按键
            if(!game_started) {
                set_start_direction(key_val);
                game_started = 1;
            } else {
                update_direction(key_val);
            }
        }

        // ----- 游戏移动（仅在已启动且未结束时）-----
        if(game_started && !game_over) {
            speed_counter++;
            if(speed_counter >= game_speed) {
                speed_counter = 0;
                move_snake();

                // ---------- 碰撞检测：游戏结束，执行动画，开启外部中断 ----------
                if(check_collision()) {
                    game_started = 0;       // 停止移动
                    game_over = 1;          // 进入游戏结束状态

                    // 1. 执行结束动画（保持蛇身，由下至上累积点亮 → 全亮）
                    game_over_animation();

                    // 2. 动画完成，开启外部中断0，等待重启指令
                    EX0 = 1;               // 允许外部中断0

                    // 3. 等待外部中断触发重启标志
                    while(!restart_flag) {
                        matrix_display();  // 持续显示全亮画面
                    }

                    // 4. 外部中断已触发，重新初始化游戏
                    game_init();
                    timer0_init();         // 重新初始化后定时器再次启动
                }

                // 食物被吃且未碰撞时生成新食物（move_snake已生成，此处可保留）
                if(food.exist == 0) {
                    generate_food();
                }
            }
        }

        // ----- 点阵显示（正常、动画、全亮均由此函数处理）-----
        matrix_display();
    }
}
```
### 阶段 1：初始化
游戏运行的第一步是完成所有“准备工作”，分为 3 部分：

1.  **外部中断 0 初始化**
    - `IT0 = 1`：设置外部中断 0 为下降沿触发（用于游戏结束后的重启按键）；
    - `EA = 1`：开启 51 单片机全局中断（必须开，否则中断不响应）；
    - `EX0 = 0`：初始关闭外部中断 0（避免游戏运行中误触发重启）。

2.  **游戏参数初始化（调用 `game_init()`）**
    - 端口初始化：点阵列端口 `COL_PORT=0xFF`、按键端口 `P2=0xFF`（准双向输入）；
    - 游戏状态：`game_started=0`（未启动）、`game_over=0`（未结束）、`restart_flag=0`（无重启请求）；
    - 蛇初始化：调用 `snake_init()`，初始长度 5，方向向右，坐标在 (4,4) 附近；
    - 食物初始化：调用 `food_init()` + `generate_food()`，生成第一个随机位置的食物。

3.  **定时器 0 初始化（调用 `timer0_init()`）**
    - 配置为 16 位自由运行模式，从 0 计数到 65535 循环，无中断；
    - 仅作为“硬件随机熵源”，为食物随机位置提供随机性（按键时捕获计数值更新 `random_seed`）。

---

### 阶段 2：主循环（while(1)）
这是游戏的核心，循环执行以下 3 个核心任务，永不退出：

#### 子任务 1：按键扫描与处理（启动 / 转向）
- 先调用 `key_scan()` 获取按键值（0 = 无按键，1 = 上 / 2 = 下 / 3 = 左 / 4 = 右）；
- 按键仅在 `!game_over`（未结束）时生效，分两种场景：

| 场景 | 处理逻辑 |
|------|----------|
| 游戏未启动（`game_started=0`） | 调用 `set_start_direction(key_val)` 设置初始方向，同时 `game_started=1`（游戏启动） |
| 游戏已启动（`game_started=1`） | 调用 `update_direction(key_val)` 更新方向（限制反向：如向右时不能直接向左） |

---

#### 子任务 2：蛇的移动控制（按速度触发）
蛇不会“实时移动”，而是按 `game_speed`（默认 45）控制移动频率，逻辑如下：

1.  `speed_counter` 每次循环 + 1，累计到 `game_speed` 时触发一次移动；
2.  调用 `move_snake()`：计算新头部坐标 → 检测撞墙 → 移动身体（吃食物则加长） → 更新头部；
3.  移动后立即调用 `check_collision()` 检测碰撞：  
    - 若碰撞（撞墙 / 撞自身），执行游戏结束流程：  
      ① `game_started=0`（停止移动）、`game_over=1`（标记结束）；  
      ② 调用 `game_over_animation()`：执行“由下至上逐行点亮 → 全屏亮”的结束动画；  
      ③ `EX0=1`：开启外部中断 0（允许按重启按键）；  
      ④ 进入死循环 `while(!restart_flag)`：持续显示全亮画面，等待重启中断；  
      ⑤ 中断触发（`restart_flag=1`）：调用 `game_init()` 重新初始化游戏，回到初始状态。  
4.  若食物被吃（`food.exist=0`），调用 `generate_food()` 生成新食物（避免食物位置与蛇身重叠）。  

---

#### 子任务 3：点阵显示（持续刷新）  
- 每次循环都调用 `matrix_display()`，实现画面的“动态刷新”；  
- 显示逻辑分 3 种状态：   
  - 游戏正常运行：显示蛇身（各节坐标） + 食物（随机坐标）；  
  - 游戏结束动画中：按 `lit_rows_mask` 累积点亮行（由下至上）；  
  - 动画完成后：全屏点亮（`row_data=0xFF`）。  

---

### 阶段 3：中断处理（游戏结束后的重启）
当游戏结束并开启 `EX0=1` 后，外部中断 0 触发时：  

1.  进入 `int0_isr()` 中断服务函数；  
2.  置位 `restart_flag=1`（重启请求）；  
3.  立即关闭 `EX0=0`（避免重复触发）；  
4.  主循环中检测到 `restart_flag=1`，调用 `game_init()` 重启游戏。  

---

### 完整运行流程示例（一次游戏周期）  
1.  上电 → 初始化：蛇初始位置 (4,4)~(0,4)，食物随机生成，游戏未启动；  
2.  按上键 → `game_started=1`，蛇初始方向向上；  
3.  主循环累计 `speed_counter=45` → 调用 `move_snake()`，蛇头上移一位；
4.  持续移动：按上 / 下 / 左键转向（不反向），吃到食物则加长 + 得分 + 新食物；
5.  撞墙 / 撞自身 → 触发游戏结束：
    - 执行结束动画（逐行点亮 → 全屏亮）；
    - 开启外部中断，等待重启按键；
6.  按重启按键 → 中断触发 → `game_init()` 重新初始化 → 回到步骤 1，开始新游戏。

---



# 电子锁代码

## 硬件连接

| 硬件模块 | 引脚 | 说明 |
|---------|------|------|
| LCD1602 数据口 | P0 | 8 位并行数据 |
| LCD1602 RS | P2^2 | 寄存器选择，0: 命令，1: 数据 |
| LCD1602 RW | P2^1 | 读写选择，0: 写，1: 读（本程序只写） |
| LCD1602 E | P2^0 | 使能信号，下降沿执行命令 |
| 绿色 LED | P2^3 | 密码正确时点亮 |
| 红色 LED | P2^4 | 密码错误或锁定时点亮 |
| 蜂鸣器 | P2^5 | 错误或锁定时发声 |
| 矩阵键盘 | P3 | 4×4 键盘，P3 低 4 位输出列扫描，高 4 位输入行检测 |

## 定义与常量

### 按键键值定义

```c
#define KEY_0      0
#define KEY_1      1
#define KEY_2      2
#define KEY_3      3
#define KEY_4      4
#define KEY_5      5
#define KEY_6      6
#define KEY_7      7
#define KEY_8      8
#define KEY_9      9
#define KEY_STAR   10  // * 键
#define KEY_POUND  11  // # 键
#define KEY_BACK   12  // 退格（通常用 * 键代替，但这里单独定义）
#define KEY_OK     13  // 确认（通常用 # 键代替）
#define KEY_NONE   255
```
将矩阵键盘的每个按键映射为一个数字，方便后续处理。其中 KEY_BACK 和 KEY_OK 是逻辑功能键，实际硬件上可能由 `*` 和 `#` 键实现。  
### 系统状态定义
```c
#define STATE_INPUT_PASSWORD    0  // 输入密码状态
#define STATE_CHANGE_PASSWORD   1  // 修改密码状态
#define STATE_VERIFY_PASSWORD   2  // 验证密码状态
#define STATE_LOCKED           3  // 锁定状态
```
系统有四个主要状态，通过状态机切换。  
### LED和蜂鸣器控制宏
```c
#define GREEN_LED_ON()   LED_GREEN = 1
#define RED_LED_ON()     LED_RED = 1
#define BUZZER_ON()      BUZZER = 1
...
```
简单宏定义，方便控制。  
## 全局变量
```c
// LCD相关
unsigned char g_inputPos = 0;  // 密码输入位置（用于显示星号）

// 键盘相关
static unsigned char g_lastKey = KEY_NONE;  // 上次按键值，用于消抖
static unsigned int g_keyPressTime = 0;     // 按键按下时间，用于长按检测

// 系统相关
unsigned char g_systemState = STATE_INPUT_PASSWORD;  // 当前系统状态
unsigned char g_password[4] = {1, 2, 3, 4};          // 当前密码 (默认1234)
unsigned char g_inputBuffer[4] = {0};                // 输入缓冲区
unsigned char g_inputIndex = 0;                      // 当前输入位置
unsigned char g_errorCount = 0;                       // 连续错误次数
unsigned char g_isFirstInput = 1;                     // 修改密码时，标记是第一次输入还是第二次确认
unsigned char g_newPassword[4] = {0};                 // 新密码临时缓存
```
## LCD1602驱动函数
### 延时函数
```c
void LCD_Delay(unsigned int ms) {
    unsigned int i, j;
    for(i = 0; i < ms; i++)
        for(j = 0; j < 132; j++);
}
```
针对12MHz晶振优化的毫秒级延时，用于LCD时序。   
### 写命令/数据
```c
void LCD_WriteCmd(unsigned char cmd) {
    LCD_RS = 0; // 命令模式
    LCD_RW = 0; // 写入模式
    LCD_E = 0;
    LCD_DATA = cmd;
    LCD_E = 1;  // 产生使能脉冲
    LCD_Delay(1);
    LCD_E = 0;
    LCD_Delay(2);
}

void LCD_WriteData(unsigned char dat) {
    LCD_RS = 1; // 数据模式
    LCD_RW = 0;
    LCD_E = 0;
    LCD_DATA = dat;
    LCD_E = 1;
    LCD_Delay(1);
    LCD_E = 0;
    LCD_Delay(2);
}
```
标准LCD1602写操作，先设置RS/RW，然后送数据，最后产生E脉冲。  
### LCD初始化
```c
void LCD_Init(void) {
    LCD_Delay(15);
    LCD_WriteCmd(0x38); // 8位数据，2行显示，5x7点阵
    LCD_Delay(5);
    LCD_WriteCmd(0x38);
    LCD_Delay(1);
    LCD_WriteCmd(0x38);
    LCD_WriteCmd(0x08); // 显示关闭
    LCD_WriteCmd(0x01); // 清屏
    LCD_WriteCmd(0x06); // 光标移动设置：写入后光标右移
    LCD_WriteCmd(0x0C); // 显示开，光标关，闪烁关
}
```
按照LCD1602手册要求的上电初始化序列。  
### 光标设置与显示函数
```c
void LCD_SetCursor(unsigned char x, unsigned char y) {
    unsigned char addr;
    if(y == 0) addr = 0x80 + x;
    else addr = 0xC0 + x;
    LCD_WriteCmd(addr);
}

void LCD_ShowChar(unsigned char x, unsigned char y, unsigned char ch) {
    LCD_SetCursor(x, y);
    LCD_WriteData(ch);
}

void LCD_ShowStr(unsigned char x, unsigned char y, unsigned char *str) {
    LCD_SetCursor(x, y);
    while(*str != '\0') {
        LCD_WriteData(*str);
        str++;
    }
}
```
提供了光标定位和显示字符/字符串的函数。  
### 密码显示专用函数
```c
void LCD_AddStar(void) {
    if(g_inputPos < 16) {
        LCD_ShowChar(g_inputPos, 1, '*');
        g_inputPos++;
    }
}

void LCD_DelStar(void) {
    if(g_inputPos > 0) {
        g_inputPos--;
        LCD_ShowChar(g_inputPos, 1, ' ');
        LCD_SetCursor(g_inputPos, 1);
    }
}
```
这两个函数用于在LCD第二行显示密码星号，并维护当前输入位置g_inputPos。每输入一位显示一个*，退格则清除最后一个星号。  

## 矩阵键盘驱动
### 键盘扫描
```c
unsigned char Keypad_Scan(void) {
    unsigned char col, key_value = KEY_NONE;
    unsigned char temp;
    // 逐行扫描
    KEYPAD_PORT = 0xFE; // 第一行输出低电平
    temp = KEYPAD_PORT & 0xF0;
    if(temp != 0xF0) {
        // 消抖后读取列值
        ...
    }
    // 类似处理第二行(0xFD)、第三行(0xFB)、第四行(0xF7)
    ...
    return key_value;
}
```
- 4行（输出）：通过 `KEYPAD_PORT` 低4位控制，依次拉低某一行，其余行置高；
- 4列（输入）：通过 `KEYPAD_PORT` 高4位检测，若某列被拉低，说明对应行列交叉处的按键按下。  

函数中按键编码与行列的对应关系（4行×4列=16个按键）：

| 行号 | 行控制值（KEYPAD_PORT） | 列0（0xE0） | 列1（0xD0） | 列2（0xB0） | 列3（0x70） |
|------|-------------------------|-------------|-------------|-------------|-------------|
| 第1行 | 0xFE（二进制11111110）| 按键0       | 按键1       | 按键2       | 按键3       |
| 第2行 | 0xFD（二进制11111101）| 按键4       | 按键5       | 按键6       | 按键7       |
| 第3行 | 0xFB（二进制11111011）| 按键8       | 按键9       | 按键10      | 按键11      |
| 第4行 | 0xF7（二进制11110111）| 按键12      | 按键13      | 按键14      | 按键15      |  
- - -
以第一行为例子
```c
// 扫描第一行
KEYPAD_PORT = 0xFE;  // 拉低第1行（低4位第0位为0），其余行置高
temp = KEYPAD_PORT;
temp &= 0xF0;        // 仅保留高4位（列输入值），屏蔽低4位（行输出值）

if(temp != 0xF0) {   // 高4位不全为1 → 有列被拉低 → 可能有按键按下
    LCD_Delay(10);   // 10ms消抖：过滤机械按键的抖动干扰
    temp = KEYPAD_PORT & 0xF0;  // 消抖后再次读取列值
    
    if(temp != 0xF0) {  // 再次确认有按键按下（排除抖动）
        // 根据列值判断具体列号
        switch(temp) {
            case 0xE0: col = 0; break;  // 高4位1110 → 第0列按下
            case 0xD0: col = 1; break;  // 高4位1101 → 第1列按下
            case 0xB0: col = 2; break;  // 高4位1011 → 第2列按下
            case 0x70: col = 3; break;  // 高4位0111 → 第3列按下
            default: return KEY_NONE;   // 异常值，直接返回无按键
        }
        // 计算按键编码：第1行基准值0 + 列号
        key_value = 0 + col;
    }
}
```
后续三行仅行号基准值、编码不同  

### 带消抖的按键获取
```c
unsigned char Keypad_GetKey(void) {
    unsigned char currentKey = Keypad_Scan();
    if(currentKey != KEY_NONE) {
        if(g_lastKey == KEY_NONE) {
            g_lastKey = currentKey; // 第一次检测到，记录但不返回
            return KEY_NONE;
        }
        else if(g_lastKey == currentKey) {
            return currentKey;       // 连续两次相同，确认按键
        }
    }
    else {
        g_lastKey = KEY_NONE;        // 无按键，重置
    }
    return KEY_NONE;
}
```
实现简单的按键消抖：只有当连续两次扫描到同一个按键时才认为有效，返回键值。这样可以避免机械抖动造成的误触发。  
### 长按检测
```c
void UpdateKeyDisplay(void) {
    static unsigned char lastScanKey = KEY_NONE;
    static unsigned int keyHoldTime = 0;
    unsigned char currentScanKey = Keypad_Scan();

    if(currentScanKey != lastScanKey) {
        lastScanKey = currentScanKey;
        keyHoldTime = 0;
    } else if(currentScanKey != KEY_NONE) {
        keyHoldTime++;
        if(keyHoldTime > 100) {          // 长按约1秒（假设主循环10ms）
            if(currentScanKey == KEY_OK) {
                // 进入修改密码模式
                g_systemState = STATE_CHANGE_PASSWORD;
                ...
            }
        }
    } else {
        lastScanKey = KEY_NONE;
        keyHoldTime = 0;
    }
}
```
`UpdateKeyDisplay`函数在主循环中频繁调用，用于检测按键长按事件。当检测到`KEY_OK`（通常是`#`键）长按超过100个循环（约1秒）时，系统切换到修改密码状态。  
## LED和蜂鸣器控制
```c
void AlarmShort(void) {
    unsigned int i;
    for(i = 0; i < 500; i++) {
        BUZZER = ~BUZZER;
        DelayMS(1);
    }
    BUZZER = 0;
}
```
短报警：蜂鸣器以1ms间隔翻转，持续500次，即产生约1秒的1kHz方波声音。   
```c
void AlarmLong(void) {
    unsigned int i;
    for(i = 0; i < 1500; i++) {
        BUZZER = ~BUZZER;
        DelayMS(1);
    }
    BUZZER = 0;
}
```
长报警：持续3秒，用于锁定警告。  
```c
void LED_Flashing(unsigned char times, unsigned int interval) {
    unsigned char i;
    for(i = 0; i < times; i++) {
        ALL_LED_ON();
        DelayMS(interval);
        ALL_LED_OFF();
        DelayMS(interval);
    }
}
```
LED闪烁效果，用于提示密码修改成功。  

## 系统核心函数
### 重置输入缓冲区
```c
void ResetInputBuffer(void) {
    unsigned char i;
    for(i = 0; i < 4; i++) {
        g_inputBuffer[i] = 0;
    }
    g_inputIndex = 0;
    LCD_ResetPasswordDisplay();
}
```
清除输入缓冲区，并将LCD上的星号清除  
### 密码验证与比较
```c
unsigned char CheckPassword(void) {
    unsigned char i;
    for(i = 0; i < 4; i++) {
        if(g_inputBuffer[i] != g_password[i]) {
            return 0;
        }
    }
    return 1;
}

void CopyPassword(unsigned char *dest, unsigned char *src) {
    unsigned char i;
    for(i = 0; i < 4; i++) {
        dest[i] = src[i];
    }
}

unsigned char ComparePassword(unsigned char *pwd1, unsigned char *pwd2) {
    unsigned char i;
    for(i = 0; i < 4; i++) {
        if(pwd1[i] != pwd2[i]) {
            return 0;
        }
    }
    return 1;
}
```
简单的数组比较和复制函数。  

## 状态机处理函数- `main`
```c
void main(void) {
    // 初始化所有模块
    LCD_Init();
    Keypad_Init();
    LED_Buzzer_Init();
    
    // 显示欢迎信息
    LCD_ShowStr(0, 0, "Electronic Lock");
    LCD_ShowStr(0, 1, "System Ready");
    // 【删除】移除了 DisplayKeyValue(KEY_NONE);
    
    DelayS(2);
    
    // 主循环
    while(1) {
        switch(g_systemState) {
            case STATE_INPUT_PASSWORD:
                StateInputPassword();
                break;
                
            case STATE_VERIFY_PASSWORD:
                StateVerifyPassword();
                break;
                
            case STATE_LOCKED:
                StateLocked();
                break;
                
            case STATE_CHANGE_PASSWORD:
                StateChangePassword();
                break;
                
            default:
                g_systemState = STATE_INPUT_PASSWORD;
                break;
        }
        
        DelayMS(10);
    }
}
```
上电后首先执行硬件模块初始化，为系统运行做准备：  
- LCD1602 初始化
- 按键矩阵初始化
- LCD1602 初始化
然后显示欢迎界面  
```c
// 显示欢迎信息
LCD_ShowStr(0, 0, "Electronic Lock");
LCD_ShowStr(0, 1, "System Ready");
DelayS(2);
```
然后进入状态机主循环    
系统有四个状态，每个状态对应一个处理函数，它们会在主循环中被调用。状态之间通过修改`g_systemState`进行切换。  
### 状态0：输入密码- `StateInputPassword`
- 显示提示"Enter Password:"，第二行等待输入。
- 循环扫描按键：
    - 数字键（0-9）：如果输入未满4位，存入缓冲区并显示星号。
    - 退格键（KEY_BACK）：删除最后一位。
    - 确认键（KEY_OK）：当输入满4位时，切换到状态3（验证密码）。
    - 星号键（KEY_STAR）：直接进入状态1（修改密码），不需要长按。
- 同时不断调用`UpdateKeyDisplay`检测长按事件（长按OK也可进入修改密码）。  
### 状态3：验证密码- `StateVerifyPassword`
- 调用`CheckPassword`比较输入和存储密码。
- 若正确：
    - 显示"Correct! Welcome!"
    - 绿灯亮1秒
    - 错误计数清零
- 若错误：
    - 显示"Error! Try Again!"
    - 红灯亮1秒，蜂鸣器短鸣
    - 错误计数加1
    - 如果错误次数达到3次，切换到状态2（锁定）
- 最后重置输入缓冲区，返回状态0（输入密码）。
### 状态2：锁定- `StateLocked`
- 显示"LOCKED! Wait 3 sec"
- 蜂鸣器长鸣3秒
- 延时3秒（实际通过循环DelayS实现）
- 重置错误计数和输入缓冲区，返回状态0。

### 状态1：修改密码- `StateChangePassword`
这个状态较为复杂，分为两个阶段：  
- __第一阶段（第一次输入）__：显示"New Password:"，用户输入4位新密码，按OK后存入`g_newPassword`，然后清屏显示"Input Again:"，进入第二阶段。
- __第二阶段（第二次确认）__：用户再次输入4位密码，按OK后与`g_newPassword`比较：
    - 如果一致，则更新`g_password`为新密码，显示"Password Changed!"，LED闪烁3次，延时1秒后返回状态0。
    - 如果不一致，显示"Not Match! Try Again"，短鸣，然后返回第一阶段重新输入。
- 任何时候可按退格键删除，按OK确认（输入满4位才有效）。
- 在修改密码过程中，长按OK检测仍然有效，但此时系统已在修改状态，不会重复进入。


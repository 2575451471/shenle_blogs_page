---
publish: true
tags:
  - 代码小记
---
﻿# 引入
前段时间考马原    
舍友给了题库的word文档， 
文档里面是题目和答案一并给出的  
想把答案遮住，点开才能看的形式，这样复习会更加高效一点  
怎么做呢，第一时间想到的是用vim  
>vim给我的首印象就是能做到更加灵活的全局搜索替换   

# 解决
1. 首先观察到文件里面近乎所有的答案行都包含了“正确答案”这一关键字
在“正确答案”和 “:C”之间插入部分html代码，得到 
`<details><summary>正确答案</summary>：C</details>`
即可实现对答案隐藏

2. 在“正确答案”前插入`<details><summary>`，在“正确答案”后插入`</summary>`
```vim
:%s/^\(.*正确答案：\)/<details><summary>\1<\/summary>/g
```
>解释：  
>>:%s：表示全局替换。  
>>^\(.*正确答案：\)：匹配每行开头到“正确答案：”部分，将其捕获到分组 \1 中。  
>>`<details><summary>\1<\/summary>`：在匹配内容前增加 `<details><summary>`，并在“正确答案：”后增加 `</summary>`。  
>>g：全局替换。  

3. 在每个含有“正确答案：”的行末尾追加 `</details>`：
```vim
:%s/\(正确答案：.*\)$/\1<\/details>/g
```
>解释：
>>\(正确答案：.*\)$：匹配“正确答案：”及其后所有内容，将其捕获到分组 \1 中。  
>>\1<\/details>：在匹配内容末尾追加 `</details>`。  

# 实现效果![img](3084836-20250107234906557-785962804.png))![img](3084836-20250107234921291-67244324.png))

# 追加
线上题库测试本来说要在考试前两天关闭，我那会刷的还不多，本来想配合这个能隐藏答案的文档写一个能随机出题的程序来着，但是文件处理这块貌似来不及学，在考前再浪费时间就有点得不偿失，就简单写了个抽取随机数对应这个文档的题号的c。  
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define TOTAL_SINGLE 40
#define TOTAL_MULTIPLE 20
#define TOTAL_TRUE_FALSE 20
#define MAX_CHAPTERS 8

// 各章节的题目数量
const int singleCounts[MAX_CHAPTERS] = {26, 188, 117, 117, 145, 32, 45, 22};
const int multipleCounts[MAX_CHAPTERS] = {26, 106, 119, 116, 87, 76, 59, 27};
const int trueFalseCounts[MAX_CHAPTERS] = {18, 114, 77, 54, 43, 50, 37, 49};

// 记录得分
int score = 0;

// 随机生成一个题目
void generateQuestion(int type, int chapter, int question, int currentIndex) {
    const char *typeStr;
    if (type == 1) typeStr = "dan"; // 单选
    else if (type == 2) typeStr = "duo"; // 多选
    else typeStr = "pan"; // 判断

    printf("试卷第%d题: %d.%s.%d\n", currentIndex, chapter, typeStr, question);
}

// 从某个题库中随机抽取题目
int getRandomQuestion(int totalQuestions) {
    return rand() % totalQuestions + 1;
}

int main() {
    srand(time(NULL)); // 初始化随机种子

    int usedSingle = 0, usedMultiple = 0, usedTrueFalse = 0;
    int selectedChapters[MAX_CHAPTERS] = {0}; // 用于记录抽取的章节

    printf("随机试卷生成开始\n\n");

    int currentIndex = 1; // 当前是第几题

    // 随机生成单选题
    for (int i = 0; i < TOTAL_SINGLE; i++, currentIndex++) {
        int chapter = rand() % MAX_CHAPTERS; // 随机选择章节
        int question = getRandomQuestion(singleCounts[chapter]);
        generateQuestion(1, chapter, question, currentIndex);

        printf("请输入答案是否正确(1错误/2正确，3提前交卷): ");
        int isCorrect;
        scanf("%d", &isCorrect);

        if (isCorrect == 3) {
            printf("提前交卷，未作答的题目将按错误处理。\n");
            goto finish;
        }

        if (isCorrect == 2) score += 1; // 单选题每题1分
    }

    // 随机生成多选题
    for (int i = 0; i < TOTAL_MULTIPLE; i++, currentIndex++) {
        int chapter = rand() % MAX_CHAPTERS; // 随机选择章节
        int question = getRandomQuestion(multipleCounts[chapter]);
        generateQuestion(2, chapter, question, currentIndex);

        printf("请输入答案是否正确(1错误/2正确，3提前交卷): ");
        int isCorrect;
        scanf("%d", &isCorrect);

        if (isCorrect == 3) {
            printf("提前交卷，未作答的题目将按错误处理。\n");
            goto finish;
        }

        if (isCorrect == 2) score += 2; // 多选题每题2分
    }

    // 随机生成判断题
    for (int i = 0; i < TOTAL_TRUE_FALSE; i++, currentIndex++) {
        int chapter = rand() % MAX_CHAPTERS; // 随机选择章节
        int question = getRandomQuestion(trueFalseCounts[chapter]);
        generateQuestion(3, chapter, question, currentIndex);

        printf("请输入答案是否正确(1错误/2正确，3提前交卷): ");
        int isCorrect;
        scanf("%d", &isCorrect);

        if (isCorrect == 3) {
            printf("提前交卷，未作答的题目将按错误处理。\n");
            goto finish;
        }

        if (isCorrect == 2) score += 1; // 判断题每题1分
    }

finish:
    // 剩余未答题按错误处理
    score += (TOTAL_SINGLE + TOTAL_MULTIPLE + TOTAL_TRUE_FALSE - currentIndex + 1) * 0;

    printf("\n试卷完成，您的总得分是: %d 分\n", score);

    return 0;
}

```
* 效果![img](3084836-20250107235813747-1869493674.png))
>结果被坑了，题库最后一天前晚上一点多才关上的，折腾这些没用上捏  
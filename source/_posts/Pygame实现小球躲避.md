---
title: Pygame实现小球躲避
categories: Python
keywords: PyGame
top_img: /img/default.webp
cover: /img/pygame.webp
abbrlink: 72f74028
date: 2021-12-18 20:08:26
---

##  前言：

这学期的Python课，要写代码是真的多...

课程实验一是一个五子棋，但是发了代码。

至于代码质量嘛～  我直接全部根据自己划分的结构改了

这里吐槽下 (真的发下来的代码 惨不忍睹 ) 

我改了快4个小时 后面功能不想加了...

这次是自己写嘛～ 那就写个想样的。

##  结构划分

我分为了

- run  入口
- setting  设置
- main 主逻辑
- utils 仓库

其实我想的是：全部设置到页面上去，但是偷懒～ （期末要去弄绩点）

直接开始贴代码

###  run.py

```python
import sys

from main import main

banner = """ ____        _ _ _____          
| __ )  __ _| | | ____|___  ___ 
|  _ \ / _` | | |  _| / __|/ __|
| |_) | (_| | | | |___\__ \ (__ 
|____/ \__,_|_|_|_____|___/\___|
"""

if __name__ == '__main__':
    print(banner)
    print("Author: HengYi")
    print("[*] 简单：输入 1")
    print("[*] 普通：输入 2")
    print("[*] 困难：输入 3")
    try:
        num = int(input("请选择难度："))
        if num in [1, 2, 3]:
            main(num)
        else:
            print("无法处理～")
            sys.exit()
    except Exception as e:
        raise Exception("无法处理～")
```

###  setting.py

```python
WIDTH = 900  # 宽
HEIGHT = 600  # 高
SCORE = 0  # 分数
TIME = 0  # 时间
FIRST_STEP = 10  # 到达第二关时间
SECOND_STEP = 20  # 到达第三关时间
FPS = 60  # 刷新率
BG_COLOR = (255, 239, 213)  # 背景颜色
```

###  utils.py

```python
# -*- coding: utf-8 -*-
import pygame
from setting import FIRST_STEP, SECOND_STEP, BG_COLOR, WIDTH, HEIGHT


# Note: 根据难度生成对应的小球
# Time: 2021/12/17 8:35 下午
# Author: HengYi
def ballNum(ladderNum, time):
    index = 0
    if FIRST_STEP <= time < SECOND_STEP:
        index = 1
    if SECOND_STEP <= time:
        index = 2
    numMap = [
        [2, 3, 5],
        [3, 5, 6],
        [4, 6, 7]
    ]
    return numMap[ladderNum - 1][index]


# Note: 根据小球个数设置防止误触时间
# Time: 2021/12/17 8:43 下午
# Author: HengYi
def protectTime(ballsNum):
    if ballsNum in [2, 3, 4]:
        return 1
    else:
        return 2


# Note: 根据时间设置小球大小
# Time: 2021/12/17 8:58 下午
# Author: HengYi
def howBigBallIs(ladderNum, time):
    index = 0
    if FIRST_STEP <= time < SECOND_STEP:
        index = 1
    if SECOND_STEP <= time:
        index = 2
    numMap = [
        [25, 20, 15],
        [24, 20, 16],
        [26, 20, 16]
    ]
    return numMap[ladderNum - 1][index]


# Note: 根据时间难度计算球体的大小和速度
# Time: 2021/12/17 9:15 下午
# Author: HengYi
def judgeDiff(ladderNum, time):
    index = 0
    if FIRST_STEP <= time < SECOND_STEP:
        index = 1
    if SECOND_STEP <= time:
        index = 2
    numMap = [
        [(30, 30, 3.5, 3.5), (28, 28, 6, 6), (26, 26, 9, 9)],
        [(30, 30, 4.5, 4.5), (27, 27, 8, 8), (25, 25, 10, 10)],
        [(30, 30, 5, 5), (26, 26, 9, 9), (24, 24, 12, 12)]
    ]
    return numMap[ladderNum - 1][index]


# Note: 创建平台窗口
# Time: 2021/12/17 2:58 下午
# Author: HengYi
def makeGameBg(width, height):
    pygame.init()
    screen = pygame.display.set_mode((width, height))  # 设置窗口大小
    pygame.display.set_caption('小球逃逃逃')  # 设置窗口标题
    background = pygame.Surface(screen.get_size())  # 填充背景
    return screen, background


# Note: 添加小球产生的事件
# Time: 2021/12/17 3:06 下午
# Author: HengYi
def ballCome():
    COME_AGAIN = pygame.USEREVENT
    pygame.time.set_timer(COME_AGAIN, 1000)
    return COME_AGAIN


# Note: 提示字体
# Time: 2021/12/17 3:11 下午
# Author: HengYi
def makeTips(content, size, color, position, screen):
    font = pygame.font.SysFont('arial', size)
    text_sf = font.render(content, True, color, BG_COLOR)
    screen.blit(text_sf, position)


# Note: 字体展示
# Time: 2021/12/18 4:20 下午
# Author: HengYi
def draw(screen, SCORE, TIME):
    screen.fill(BG_COLOR)  # 防止出现拖影
    makeTips('SCORE: ', 30, (34, 139, 34), (5, 40), screen)
    makeTips('TIME(s): ', 30, (64, 158, 255), (5, 75), screen)
    makeTips(str(int(SCORE)), 30, (34, 139, 34), (135, 40), screen)
    makeTips(str(int(TIME)), 30, (64, 158, 255), (135, 75), screen)
    if TIME in [FIRST_STEP, FIRST_STEP + 1]:
        makeTips('Ops! LEVEL_2~', 30, (60, 179, 113), (WIDTH / 2 - 30 * 3.5, HEIGHT / 2 - 50), screen)
    elif TIME in [SECOND_STEP, SECOND_STEP + 1]:
        makeTips('Congratulations! LEVEL_3', 25, (60, 179, 113), (WIDTH / 2 - 25 * 6.25, HEIGHT / 2 - 50), screen)
```

###  Main.py

```python
# -*- coding: utf-8 -*-
import random
from setting import *
from utils import *


class Ball(pygame.sprite.Sprite):
    def __init__(self, *keys):  # 创建球
        super().__init__()
        self.timeSec = 0
        w, h, xs, ys = keys[0]
        self.w = w
        self.h = h
        self.xs = xs
        self.ys = ys
        x = random.randint(0, WIDTH - self.w)
        y = random.randint(0, HEIGHT - self.h)
        self.rect = pygame.Rect(x, y, self.w * 2, self.h * 2)

    def update(self, screen, *args):
        # 根据设置的速度进行运动
        self.rect.x += self.xs
        self.rect.y += self.ys
        # 当遇到墙的时候进行反弹
        if self.rect.x > WIDTH - self.w or self.rect.x < 0:
            self.xs = -self.xs
        elif self.rect.y > HEIGHT - self.h or self.rect.y < 0:
            self.ys = -self.ys
        if self.timeSec <= args[0]:
            pygame.draw.rect(screen, (100, 149, 237), [self.rect.x, self.rect.y, self.rect.w, self.rect.h], 2)
        pygame.draw.circle(screen, (255, 0, 0), [self.rect.center[0], self.rect.center[1]], self.w)

    def timerAdd(self):
        self.timeSec += 1
        return self.timeSec

    def __del__(self):  # 销毁的时候
        pass


class Mouse(pygame.sprite.Sprite):
    def __init__(self, *keys):
        super().__init__()
        self.size = keys[0]  # 设置圆的大小
        self.rect = pygame.Rect(WIDTH / 2 - self.size, HEIGHT / 2 - self.size, self.size * 2, self.size * 2)  # 实则是一个正方形

    def update(self, screen, *args):
        if pygame.mouse.get_focused():  # 如果存在于页面内
            self.rect.center = pygame.mouse.get_pos()
        # 限制球不能半身跑到边框上
        if self.rect.x < 0:
            self.rect.x = 0
        elif self.rect.x > WIDTH - self.rect.w:
            self.rect.x = WIDTH - self.rect.w
        elif self.rect.y < 0:
            self.rect.y = 0
        elif self.rect.y > HEIGHT - self.rect.h:
            self.rect.y = HEIGHT - self.rect.h
        pygame.draw.circle(screen, (255, 0, 0), [self.rect.center[0], self.rect.center[1]], self.size)  # 根据圆心画圆


def main(ladderNum):
    # -------------------画布初始化-----------------------
    screen, background = makeGameBg(WIDTH, HEIGHT)
    clock = pygame.time.Clock()
    comeAgain = ballCome()
    # --------------------------------------------------

    # --------------------对象存储-------------------------
    global TIME, SCORE
    balls = pygame.sprite.Group(Ball(judgeDiff(ladderNum, TIME)))
    mouse = Mouse(howBigBallIs(ladderNum, TIME))
    mouseObject = pygame.sprite.Group(mouse)
    # --------------------------------------------------

    # ---------------------游戏主程序-----------------------
    RUNNING = True
    SHOWINFO = False
    while True:
        draw(screen, SCORE, TIME)  # 动态添加文字
        if SHOWINFO:
            makeTips('Please Press The Space To Restart', 30, (255, 99, 71), (WIDTH / 2 - 240, HEIGHT / 2 - 50),
                     screen)
        for each in balls:
            if pygame.sprite.spritecollide(each, mouseObject, False, collided=None) and each.timeSec > 2:
                RUNNING = False
                SHOWINFO = True
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit(0)
            elif event.type == pygame.KEYDOWN:  # 重新开始
                if event.key == pygame.K_SPACE:
                    SCORE = 0
                    TIME = 0
                    for each in balls.sprites():
                        balls.remove(each)
                    SHOWINFO = False
                    RUNNING = True
            elif event.type == comeAgain and RUNNING:  # 每秒增加
                TIME += 1
                ballsNum = ballNum(ladderNum, TIME)
                diff = judgeDiff(ladderNum, TIME)
                SCORE += (ballsNum * diff[3])
                if TIME in [10, 20]:
                    mouseObject.remove(mouseObject.sprites()[0])
                    mouseObject.add(Mouse(howBigBallIs(ladderNum, TIME)))
                if len(balls) < ballsNum:
                    balls.add(Ball(diff))
                for each in balls:  # 防止误触的保护罩
                    each.timerAdd()
        balls.update(screen, protectTime(ballNum(ladderNum, TIME)))
        mouseObject.update(screen)
        clock.tick(FPS)
        pygame.display.update()  # 刷新
    print('游戏结束')
```

##  总结

###  效果图：

<img src="/img/ballResult.webp" alt="效果图" style="zoom:40%;" />

###   如何食用：

把上面4处代码Copy下来在用`run.py`启动

里面设计的 如何判断；如何重来；... (我觉得我的变量名字已经够清楚了🐶)

欢迎与我交流，讨论。 

## 这是一篇充满个人感情色彩的学习过程笔记（记录了不少自己的心情
- 先说一下看到题目的初期感受:
英文什么的看不懂啦，朕的皇家翻译呢：说实话，一开始看到全英文档时，我的内心是崩溃的，我是真不爱看这洋文
......
嘛，也安慰自己做下去了
规则什么的理解不了啦：本身对这种游戏桌游之类的东西不熟悉，造成了我一定程度上对规则的不理解
......
......
终于搞懂规则啦，开动......
动不了一点:cry:发现代码注释也是全英的
没事，区区英文，能打倒朕？
我近代史纲的小组作业都是拿英语汇报的还怕这个？（稍微这样安慰自己了
不过值得高兴的一点是，项目的整体架构给定了:smile:
我一开始还以为要自己从头弄，差点两眼一黑：:smile:
做这个的时候又忘记换ok包了，折腾好久:cry:

## 做题想法及过程：
### 我理解的游戏规则：
```
游戏规则：
1. 两名玩家掷骰子,轮流,至少100分一局
2. 可自行选择投几个骰子,但不超过十个
3. pig out:只要有一个骰子点数为1,则本轮得分为1
4. free bacon:选择0个骰子,获得K+3分(其中k的取值和n(对手的总分)息息相关,n代表pi的位数,k代表该位数上的值)
5. Swine Align:某一个玩家掷完骰子后,两个玩家点数都不为0的情况下,二者的最大公因数(GCD)大于等于10,则该玩家可再掷一次
6. Pig pass:某一个玩家掷完骰子后,该玩家的点数仍小于另一个玩家,且点数差小于3,则该玩家可再掷一次
```
### 01 roll_dice
def roll_dice(num_rolls, dice=six_sided): % 默认是六面骰子呢，话说四面骰子(正四面体该怎么骰子呢(好奇))
 
    Simulate rolling the DICE exactly NUM_ROLLS > 0 times. Return the sum of
    the outcomes unless any of the outcomes is 1. In that case, return 1.

    num_rolls:  The number of dice rolls that will be made.选择投掷的骰子数
    dice:       A function that simulates a single dice roll outcome.每个骰子的结果

solution:
```
point=0
    for i in range(num_rolls+1):
        k=dice()
        if k==1:
            point =1
            return point 
        else:
            point+=k
    return point 
``` 
上面那个是初版，当时没仔细看index
测验的时候过不了
counted_dice过不了
因为上述代码在k=1时直接返回了结果
然后就改成了以下代码
增加了一个状态标记zt
```
point =0
    i=0
    zt=0
    while i<num_rolls:
        k=dice()
        if k==1:
            zt=1
            i+=1
        else:
            point+=k
            i+=1
    if zt==0:
        return point
    else:
        return 1
```
### 02 free bacon
def free_bacon(score):

    """Return the points scored from rolling 0 dice (Free Bacon).

    score:  The opponent's current score.
    """
    assert score < 100, 'The game should be over.'
    pi = FIRST_101_DIGITS_OF_PI

solution:
```
pi=pi//(10**(100-score))
```
卡在了//和**的用法
用成了c++语言
诶，我才是最需要练习的那个:cry:

### 03 take_turn
    """开一局函数Simulate a turn rolling NUM_ROLLS dice, which may be 0 (Free Bacon).
    Return the points scored for the turn by the current player.

    num_rolls:       The number of dice rolls that will be made.
    opponent_score:  The total score of the opponent.
    dice:            A function that simulates a single dice roll outcome.
    """

solution:
```
if num_rolls==0:
        return free_bacon(opponent_score)
    else:
        return roll_dice(num_rolls,dice)
```
介个没啥感想，pass

### 04a swine_align
    """Return whether the player gets an extra turn due to Swine Align.

    player_score:   The total score of the current player.
    opponent_score: The total score of the other player.

    >>> swine_align(30, 45)  # The GCD is 15.
    True
    >>> swine_align(35, 45)  # The GCD is 5.
    False
    """

solution:
```
if opponent_score==0 or player_score==0:
        return False
    else:
        while opponent_score!=0:
            t=opponent_score
            opponent_score=player_score%opponent_score
            player_score=t
        if player_score>=10:
            return True
        else:
            return False
```
辗转相除法！豪用:）

### 04b pig_pass
    """Return whether the player gets an extra turn due to Pig Pass.

    player_score:   The total score of the current player.
    opponent_score: The total score of the other player.

    >>> pig_pass(9, 12)
    False
    >>> pig_pass(10, 12)
    True
    >>> pig_pass(11, 12)
    True
    >>> pig_pass(12, 12)
    False
    >>> pig_pass(13, 12)
    False
    """

solution:
```
if player_score<opponent_score and opponent_score-player_score<3:
        return True
    else:
        return False
```

### 05&06 play 
    """Simulate a game and return the final scores of both players, with Player
    0's score first, and Player 1's score second.

    A strategy is a function that takes two total scores as arguments (the
    current player's score, and the opponent's score), and returns a number of
    dice that the current player will roll this turn.

    strategy0:  The strategy function for Player 0, who plays first.
    strategy1:  The strategy function for Player 1, who plays second.
    score0:     Starting score for Player 0
    score1:     Starting score for Player 1
    dice:       A function of zero arguments that simulates a dice roll.
    goal:       The game ends and someone wins when this score is reached.
    say:        The commentary function to call at the end of the first turn.
    """

solution:
```
while score0<goal and score1<goal:
        if who==0:
            num_rolls=strategy0(score0,score1)
            if num_rolls:
                score0+=roll_dice(num_rolls, dice) 
            else:
                score0+=free_bacon(score1)
            say=say(score0,score1)
            if extra_turn(score0,score1):
                continue  
        else:
            num_rolls=strategy1(score1,score0)
            if num_rolls:
                score1+=roll_dice(num_rolls, dice) 
            else:
                score1+=free_bacon(score0)
            say=say(score0,score1)
            if extra_turn(score1,score0):
                continue
            
        who=other(who)
```
做的时候把strategy的意思搞懂就通了

### 07 announce_highest
    """Return a commentary function that announces when WHO's score
    increases by more than ever before in the game.

    NOTE: the following game is not possible under the rules, it's just
    an example for the sake of the doctest

    >>> f0 = announce_highest(1) # Only announce Player 1 score gains
    >>> f1 = f0(12, 0)
    >>> f2 = f1(12, 9)
    9 point(s)! The most yet for Player 1
    >>> f3 = f2(20, 9)
    >>> f4 = f3(20, 30)
    21 point(s)! The most yet for Player 1
    >>> f5 = f4(20, 47) # Player 1 gets 17 points; not enough for a new high
    >>> f6 = f5(21, 47)
    >>> f7 = f6(21, 77)
    30 point(s)! The most yet for Player 1
    """

solution:
```
def say(score0,score1):
    if who:
        score=score1
    else:
        score=score0
    incre=score-last_score
    if incre>running_high:
        print(incre,"point(s)! The most yet for Player",who)
        return announce_highest(who,score,incre)    
    return announce_highest(who,score,running_high)
return say
```
初识高阶函数

### 08 make_averaged
    """Return a function that returns the average value of ORIGINAL_FUNCTION
    when called.

    To implement this function, you will have to use *args syntax, a new Python
    feature introduced in this project.  See the project description.

    >>> dice = make_test_dice(4, 2, 5, 1)
    >>> averaged_dice = make_averaged(dice, 1000)
    >>> averaged_dice()
    3.0
    """

solution:
```
def averaged_dice(*args):#*args的应用
        sum,i=0,0
        while i<trials_count:
           sum+=original_function(*args)
           i+=1
        avg=sum/trials_count
        return avg
    return averaged_dice
```
*args的运用

### 09 max_scoring_num_rolls
    """Return the number of dice (1 to 10) that gives the highest average turn
    score by calling roll_dice with the provided DICE over TRIALS_COUNT times.
    Assume that the dice always return positive outcomes.

    >>> dice = make_test_dice(1, 6)
    >>> max_scoring_num_rolls(dice)
    1
    """

solution:
```
averaged_dice=make_averaged(roll_dice,trials_count)
    i=1
    ms=0
    #find the maxscore,return the low_roll
while i<=10:
        avg=averaged_dice(i,dice)
        if avg>ms:
            max=i
            ms=avg
        i+=1
    return max
```
这道题卡在理解意思上了:cry:
是时候拿出我的有道翻译力）
不过最后靠测试用例理解了

### 10 bacon_strategy
    """This strategy rolls 0 dice if that gives at least CUTOFF points, and
    rolls NUM_ROLLS otherwise.
    """

solution:
```
score=free_bacon(opponent_score)
if score>=cutoff:
    return 0
return num_rolls
```

### 11 extra_turn_strategy
    """This strategy rolls 0 dice when it triggers an extra turn. It also
    rolls 0 dice if it gives at least CUTOFF points and does not give an extra turn.
    Otherwise, it rolls NUM_ROLLS.
    """

solution:
```
score+=free_bacon(opponent_score)
    if pig_pass(score,opponent_score) or swine_align(score,opponent_score):
        return 0
    return bacon_strategy(score,opponent_score,cutoff,num_rolls)  # Replace this statement
```

### 12 final_strategy
    """Write a brief description of your final strategy."""
solution:
```
score+=free_bacon(opponent_score)#第一局用free培根保个底
    average_dice=make_averaged(roll_dice,trials_count=1000)
    avg=average_dice(always_roll(4))#算出平均值
    return extra_turn_strategy(score,opponent_score,avg,4)#如果此局用free_bacon所得分数>=平均值
```

## 最后叨叨几句
经过12道题的折磨后
终于搞完了
有60%的解脱39%的兴奋
还有1%的不舍（坏了，成m了）
......
嘛
说正经的
通过这次任务
收获还是不少的
不管从专业能力方面
还是耐心耐性的磨炼上
:smile:

## 最后的最后，附上链接
https://github.com/ddmxy/wjwork.git
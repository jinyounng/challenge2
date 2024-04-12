# CAU56753 Challenge II: Route Search

## Problem definition / 문제정의

The second programming challenge is simpler than the first one.

두번째 프로그래밍 도전과제는 첫 과제보다는 단순합니다.

In this challenge, you have to make a local search algorithm for finding the **BEST** position of your initial villages.

이 도전과제에서, 여러분은 주어진 상황에서 초기 마을을 짓기에 가장 좋은 위치를 찾아야 합니다.

Here's PEAS description. According to the description, the system will automatically run your code and give you the evaluation result.

아래에 PEAS 상세정보가 주어져 있습니다. 평가 시스템은 PEAS 정보에 따라 여러분의 코드를 실행하고 평가하여, 평가 결과를 제공할 것입니다.

(Are you curious about how to run the evaluation? Then, go to [RUN](./#RUN) section!)
(어떻게 평가를 실행하는지 궁금하다면, [RUN](./#RUN) 부분으로 넘어가세요!)

### PEAS description

#### Performance measure (수행지표)

Presented in the order of importance in evaluation.

평가시 중요한 순서 순으로 나열됨

  1. The number of errored trials (smaller is better)

     오류가 난 코드 실행의 수 (적을 수록 좋음)

  2. The expected resource income at that village, for the next five turns (larger is better)

     Note that some of resources will be weighted more compared to the other resources, when computing the expected resource income.

     For example, a game board can emphasize the importance of brick cards twice greater than the other resource cards, by giving weight two for bricks and one for the others.
    
     You can check the expected resource income for a state by calling `board.evaluate_state(state)`.

     탐색 종료 시점에 찾은 위치에서, 향후 5턴 동안 발생할 총 자원카드 이익의 기댓값 (클 수록 좋음)

     참고로, 어떤 자원카드는 다른 자원카드에 비해서 평균값 계산시 가중치가 높을 수 있습니다.

     예를 들어서, 어떤 게임 판은 벽돌 자원을 다른 자원에 비해서 2배 가치있다고 생각하여, 벽돌 자원의 가중치를 2, 나머지 자원의 가중치를 1로 설정할 수 있습니다.

     특정 상태에서 발생할 자원카드 이익의 기댓값은 `board.evaluate_state(state)`를 호출하여 확인할 수 있습니다.

  3. The number of `board.evaluate_state()` calls (smaller is better)

     `board.evaluate_state()` 함수의 호출 횟수 (적을 수록 좋음)

  4. The maximum memory usage (smaller is better; rounded down in MB)

     The usage lower than 200MB is treated as 200MB.
  
     최대 메모리 사용량 (작을 수록 좋음; MB 단위로 버림)

     200MB 이하의 사용량은 200MB로 간주함.


**Note**: Evaluation program will automatically build a table that sort your codes' evaluation result in the order of performance measure. Also, the algorithm should finish its search procedure **within 1 minutes**.

**참고**: 평가 프로그램이 여러분 코드의 평가 결과를 수행지표 순서대로 정렬하여 표 형태로 표시해줄 예정입니다. 그리고, 알고리즘의 탐색 절차는 **1분 이내**에 끝나야 합니다.

**Note**: Also, the program should use lower than **1GB in total**, including your program code on the memory. For your information: when I tested with `default.py`, the memory usage after initial loading is 22MB.

**참고**: 또한, 프로그램은 (여러분의 코드를 포함) 최대 **1GB까지** 메모리를 사용할 수 있습니다. 참고로, `default.py`로 테스트했을 때, 상태 초기화 후 메모리 사용량은 22MB였습니다. 


#### Environment (환경)

Okay, the environment follows the initial set-up procedure in the Settlers of Catan game. Here, your agent is not the only one to settle down. Before you're making your decision, some of the players may have placed their initial settlements. So, you need to do is placing one of your initial village to increase your resource income. Note that the game board is randomly generated, and the importance (weight) of resource cards is hidden to the players.

이번 도전과제의 환경은 카탄 게임의 초기 마을 설정 과정을 따릅니다. 그러니까, 여러분의 에이전트만 초기 마을 설정을 하고 있는 것이 아니라, 다른 에이전트도 함께 하고 있습니다. 여러분이 마을의 위치를 결정하기 전에, 어떤 플레이어는 이미 게임 판에 정착 마을을 만들었을 수도 있습니다. 그러니, 여러분은 그 상태에서 여러분의 초기 정착 마을 중 1개를, 자원카드 이익이 극대화되도록 지어야 합니다. 참고로, 게임 판은 랜덤하게 생성될 예정이며, 특정 자원 카드의 가중치(중요도)는 플레이어들에게 공개되지 않습니다.

1. Other players will do nothing in the game until your search is done. You don't have to worry about the thief or a knight card.

   다른 플레이어는 여러분의 탐색 작업이 끝날때까지 아무 행동도 하지 않습니다. 도둑이나 기사단 카드를 걱정할 필요가 없습니다.

2. Note that the order of building settlement is 1-2-3-4-4-3-2-1 in the basic Catan rule. Your agent is facing one of the eight cases. So, for the latter four turns (i.e., 4-3-2-1 at the last), the agent might have a village on the board randomly.

   기본 카탄 규칙에서 정착촌 건설 순서가 1-2-3-4-4-3-2-1 이라는 점에 주의하세요. 여러분의 에이전트는 이 순서 중 한 상황을 마주하게 됩니다. 그러니까, 마지막 4개 차례인 경우엔(즉 뒤의 4-3-2-1 순서), 이미 에이전트가 랜덤하게 한 마을을 건설한 상황에서 다음 마을을 건설해야 할 수도 있습니다.

In terms of seven characteristics, the game can be classified as:

환경을 기술하는 7개의 특징을 기준으로, 게임은 다음과 같이 분류됩니다:

- Almost Fully observable (거의 완전관측가능)

  You already know everything required to decide your action, except the weight of resources.

  자원의 가중치를 제외하고, 여러분은 이미 필요한 모든 내용을 알고 있습니다.

- Single-agent (단일 에이전트)

  The other agents are actually doing nothing. So you're the only one who pursues those performance measure.

  다른 모든 에이전트는 사실 아무것도 안 합니다. 그러므로, 그 판에서 여러분만이 수행지표를 최대화하려는 유일한 에이전트입니다.

- Deterministic (결정론적)

  There's no probabilistic things or unexpected chances when deciding the next state. Everything is deterministic.

  다음 상태를 결정할 때 확률적으로 결정되지도 않고, 예상치 못한 변수도 없습니다. 모든 것은 결정되어 있습니다.

- Episodic actions (일화적 행동)

  You don't need to handle the sequence of your actions to build an initial village.

  초기 마을을 건설하기 위해서 필요한 여러분의 행동의 순서를 고민할 필요가 없습니다.

- Semi-dynamic performance (준역동적 지표)

  Note that performance metrics 1, 2 are static, but the metrics 3, 4 is dynamic metric, which changes its measure by how your algorithm works. So you need some special effort for achieving good performance measure on 3 and 4, when designing your algorithm.

  지표 1과 2는 정적이고, 지표 3과 4는 동적입니다. 특히 지표 3과 4는 여러분의 알고리즘 작동에 따라 변화하는 지표입니다. 그래서 알고리즘을 설계할 때, 지표 3/4의 변화에 신경써서 설계하는 노력이 필요합니다.

- Discrete action, perception, state and time (이산적인 행동, 지각, 상태 및 시간개념)

  All of your actions, perceptions, states and time will be discrete, although you can query about your current memory usage in the computing procedure.

  여러분의 모든 행동, 지각, 상태 및 시간 흐름은 모두 이산적입니다. 여러분이 계산 도중 메모리 사용량을 시스템에 물어볼 수 있다고 하더라도 말입니다.

- Known rules (규칙 알려짐)

  All rules basically follows the original Catan game, except the rules specified above.

  모든 규칙은 위에 작성된 예외를 제외하면 기본적으로 원래의 카탄 게임을 따릅니다.

#### Actions

You can take one of the following actions.

다음 행동 중의 하나를 할 수 있습니다.

- **VILLAGE(v)**: Build a village at a specific node `v`.

  특정한 꼭짓점 `v`에 마을 짓기

  Here, the list of applicable nodes will be given by the board. The list may be empty if you reached the maximum number of villages, i.e., three.

  마을 짓기가 가능한 꼭짓점의 목록은 board가 제공합니다. 단, 가능한 마을의 수가 최대치인 3개에 도달한 경우, 목록은 빈 리스트로 제공될 수 있습니다.

**Note**: You cannot do reverse-enginnering to find out actual importance value of resource cards. All you can do is doing local search on the board.

**참고**: 역공학적 방법으로 자원카드의 실제 가중치를 찾는 것은 허용되지 않습니다. 판 위에서 국소탐색 알고리즘을 사용하는 것만 허용됩니다.


#### Sensors

You can perceive the game state as follows:

- The board (게임 판)
  - All the place of hexes(resources), villages, roads and ports

    모든 육각형(자원), 마을, 도로, 항구의 위치

  - You can ask the board to the list of applicable actions for.

    가능한 행동에 대해서 게임판 객체에 물어볼 수 있습니다.

  - You can ask the board about the expected total resource income of current state, for five turns.  
  
    현재 상태에서 향후 5턴 동안 얻게 되는 자원카드 이익의 기댓값을 물어볼 수 있습니다.

- Your resource cards (여러분의 자원카드 목록)



## Structure of evaluation system

평가 시스템의 구조

The evaluation code has the following structure.

평가용 코드는 다음과 같은 구조를 가지고 있습니다.

```text
/                   ... The root of this project
/README.md          ... This README file
/evaluate.py        ... The entrance file to run the evaluation code
/board.py           ... The file that specifies programming interface with the board
/actions.py         ... The file that specifies actions to be called
/util.py            ... The file that contains several utilities for board and action definitions.
/agents             ... Directory that contains multiple agents to be tested.
/agents/__init__.py ... Helper code for loading agents to be evaluated
/agents/load.py     ... Helper code for loading agents to be evaluated
/agents/default.py  ... A default DFS agent 
/agents/_skeleton.py... A skeleton code for your agent. (You should change the name of file to run your code)
```

All the codes have documentation that specifies what's happening on that code (only in English).

모든 코드는 어떤 동작을 하는 코드인지에 대한 설명이 달려있습니다 (단, 영어로만).

To deeply understand the `board.py` and `actions.py`, you may need some knowlege about [`pyCatan2` library](https://pycatan.readthedocs.io/en/latest/index.html).

`board.py`와 `actions.py`를 깊게 이해하고 싶다면, [`pyCatan2` library](https://pycatan.readthedocs.io/en/latest/index.html) 라이브러리에 대한 지식이 필요할 수 있습니다.

### What should I submit?

You should submit an agent python file, which has a similar structure to `/agents/default.py`.
That file should contain a class name `Agent` and that `Agent` class should have a method named `search_for_longest_route(board)`.
Please use `/agents/_skeleton.py` as a skeleton code for your submission.

`/agents/default.py`와 비슷하게 생긴 에이전트 코드를 담은 파이썬 파일을 제출해야 합니다.
해당 코드는 `Agent`라는 클래스가 있어야 하고, `Agent` 클래스는 `decide_new_village(board)` 메서드를 가지고 있어야 합니다.
편의를 위해서 `/agents/_skeleton.py`를 골격 코드로 사용하여 제출하세요.

Also, you cannot use the followings to reduce your search time:

그리고 시간을 줄이기 위해서 다음에 나열하는 것을 사용하는 행위는 제한됩니다.

- multithreading / 멀티스레딩
- multiprocessing / 멀티프로세싱
- using other libraries other than basic python libraries. / 기본 파이썬 라이브러리 이외에 다른 라이브러리를 사용하는 행위

The TA will check whether you use those things or not. If so, then your evaluation result will be marked as zero.

조교가 여러분이 해당 사항을 사용하였는지 아닌지 확인하게 됩니다. 만약 그렇다면, 해당 평가 점수는 0점으로 처리됩니다.

## RUN

실행

To run the evaluation code, do the following:

1. (Only at the first run) Install the required libraries, by run the following code on your terminal or powershell, etc:

   (최초 실행인 경우만) 다음 코드를 터미널이나 파워쉘 등에서 실행하여, 필요한 라이브러리를 설치하세요.

    ```bash
    pip install -r requirements.txt
    ```

2. Place your code under `/agents` directory.

    여러분의 코드를 `/agents` 디렉터리 밑에 위치시키세요.

3. Execute the evaluation code, by run the following code on a terminal/powershell:

    다음 코드를 실행하여 평가 코드를 실행하세요.

    ```bash 
    python evaluate.py
    ```

    If you want to print out all computational procedure, then put `--debug` at the end of python call, as follows:

    만약, 모든 계산 과정을 출력해서 보고 싶다면, `--debug`을 파이썬 호출 부분 뒤에 붙여주세요.

    ```bash 
    python evaluate.py --debug
    ```

4. See what's happening.

    어떤 일이 일어나는지를 관찰하세요.

Note: All the codes are tested both on (1) Windows 11 (23H2) with Python 3.9.13 and (2) Ubuntu 22.04 with Python 3.10. Sorry for Mac users, because you may have some unexpected errors.

모든 코드는 윈도우 11 (23H2)와 파이썬 3.9.13 환경과, 우분투 22.04와 파이썬 3.10 환경에서 테스트되었습니다. 예측불가능한 오류가 발생할 수도 있어, 미리 맥 사용자에게 미안하다는 말을 전합니다.
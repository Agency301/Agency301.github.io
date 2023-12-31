---
layout: post
title:  "Welcome to Jekyll!"
date:   2023-07-25 23:28:26 +0900
categories: jekyll update
use_math: true
---
    

-----------------------------


1. test
{:toc}

# Symbolic Discovery of Optimization Algorithms (2023.5)

## [Symbolic Discovery of Optimization Algorithms](https://arxiv.org/abs/2302.06675)

## Introduction

이 논문은 AutoML-Zero 와 상당히 유사한 내용을 가지고 있다. 하단 링크 참고

[AutoML-Zero: Evolving Machine Learning Algorithms From Scratch](https://arxiv.org/abs/2003.03384)

AutoML-Zero는 최대한 사람의 개입 없이 ML Architecture 를 구성하는 것에 대한 논문인데, 미분같은 복잡한 연산은 제외하고 65가지의 연산, 함수(instruction이라 한다.)를 조합하여 program flow를 구성하는 것을 목표로 program space를 search하였다고 한다. 사용된 방법은 evolutionary search로,  genetic algorithm에 기반하여 instruction을 넣거나 빼고, 대체하는 등의 mutation 연산, 그리고 한 time-step(=generation)이 지날 때마다 N개의 top rated program을 parents로 하여 copy + mutation하는 절차를 통해 다양한 program을 만들어냈다고 한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/39c37380-e519-4582-94b2-3185da47edd5/Untitled.png)

본 논문도 마찬가지로 program search를 통해 최적의 optimizer를 찾고자 했다. 본 논문에서는 algorithm을 개발하는 데에 있어 program을 symbolic representation으로 사용하는 것에 대한 3가지 장점을 밝히고 있다.

(1). 어차피 계산 하려면 프로그램으로 나타내야 한다 (2). 분석하기 편하고 다른 분야로 transfer가 용이하다 (3). 프로그램의 길이를 알고리즘의 복잡도를 추정하는데 사용할 수 있어, 더 간단하고 일반적인 알고리즘을 찾는데 도움이 된다.

## Program search space designing

해당 논문에서는 program search space를 설계할 때 고려할 점을 3가지 밝힌다.

(1). 신박한 알고리즘을 얻기 위해 충분히 넓어야 한다 (2). ML의 workflow에 적합해야 한다 (3). high-level design에만 집중한다

다음은 program search를 위한 4가지 구성이다.

- `train` function

    input으로 weight $w$, gradient $g$, learning rate $lr$을 받아 weight update를 output 하는 main objective이다. 즉, search의 객체이다.
    
    historical value를 저장하는 extra variable을 둘 수 있다

- building blocks

    train function은 statement와 local variable 개수에 제약을 받지 않는다. 본 논문에서는 45개의 commom math function을 채택하였고, program의 간결성을 위해 linear interpolation function $interp(x, y, a)=(1-a)x+ay$ 등을 정의하였다.

    또한 기존의 연구에서는 반복문, 조건문 등의 complex feature를 도입하였었는데 유의미한 성능 진전이 없었으므로 본 연구에서는 제외하였다고 한다.

    차원에 맞지 않는 데이터 연산은 automatically casted되도록 했다고 한다.(ex. matrix + scalar)

- mutations and redundant statements

    3가지 mutation operation을 정의한다

    1. random function을 random location에 삽입
    2. randomly chosen statement 삭제
    3. random function의 argument를 수정

    random variable이나 constant는 normal distribution에서 추출하며, constant들은 차후에 tunable hyperparameter로 기능할 수 있다.

- infinite and sparse search space

    internal arguments, state, function의 개수에 제약을 두지 않고, mutation연산의 존재 때문에 program search space가 infinite하다. 

    high-performing program은 search space상에 sparse하게 존재하기 때문에, 이를 빠르게 search하기 위해 low-cost proxy task를 설정하여 evaluation하는 것으로 보인다.

## Efficient search techniques

high-performing algorithm이 너무 sparse하게 존재하는 문제점 때문에 본 논문에서는 각종 search technique를 도입한다.

- Evolution with warm-start and restart

    warm-start: search acceleration을 위해 AdamW에서 시작

    restart: 

    - exploration: start from initial program
    - exploitation: start from best program
- Pruning through abstract execution

    별 건 아니고 에러 나는 거 없애주고, program 마다 hash값 만들어서 똑같은 program 삭제하는 것

- Proxy tasks and search cost

    low-cost proxy task로 search 진행. funnel selection과 연관.

## Program selection methodology

매우 큰 program space를 search하기 때문에 proxy task를 필연적으로 이용하게 되는데, proxy task와 target task의 차이, 그리고 다양한 domain에 적용될 수 있는 generalization 능력을 원하기 때문에, 학습과 활용에 큰 차이가 발생한다고 밝히고 있다. 이에 본 논문에서는 2가지 해결책을 사용한다.

- Funnel selection

    funnel selection은 proxy task를 통해 generalization power를 검증하는 방법이다.

    - proxy task A를 통해 problem set $S_1$을 평가한다 → selection → $S_2$
    - A보다 10배 더 큰 task B로 $S_2$의 평가를 진행한다 → selection → $S_3$
    - $S_3$를 A보다 100배 더 큰 task C로 평가한다 → selection

    이러한 과정을 통해 generalization performance가 떨어지는 program을 낙오시킨다.

- Simplification

    더 간단할 수록 generalization 능력이 높을 것이라는 직관에 의해, input과 output이 identical하거나, 없어져도 거의 영향을 미치지 않는 statement, function을 제거한다.

## Proposition: Lion (EvoLved Sign Momentum) Optimizer

~~너무 억지 작명인 것 같다~~

해당 논문에서 찾아낸 Lion Opimizer이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a1890379-340a-46e6-9c12-fee66350fcf4/Untitled.png)

- Sign update and regularization

    sign 함수를 이용하기 때문에, 모든 dimension에서 uniform magnitude를 가진다고 한다. 이 특성이 noise를 일으켜 regularization 처럼 기능한다고 설명한다. 

    cf. sign 함수가 noise를 일으키는 이유? → Lion은 RMSprop이나 Adam처럼 EMA를 사용하는데(linear interpolation 부분), 여기에 sign을 취해버려서 magnitude는 반영을 안하고 tendency만 반영하게 되어서 update가 non-smooth하게 되어서 noisy하다는 것 같음

- Momentum tracking

    $\beta_1, \beta_2$를 이용하는데, 2개를 써야하는 이유는 appendix의 ablation study에서 증명했다.(사실 evolutionary search로 찾아낸 것이기 때문에 사후 정당화에 가깝다)

- Hyperparameter and batch size choices

    Lion Optimizer의 성능은 Batch size가 커질 수록 비례하여 증가한다.

- Memory and runtime benefits

    task에 따라 AdamW 보다 2-15% 가량의 속도 향상을 보였다.

## Evaluation of Lion

ViT, LiT, Autoregressive Model 등 다양한 task에 적용했는데 AdamW보다 항상 낫거나 비슷했다고 한다. benchmark table이 너무 많으니까 논문 참조.


# AtlasYang (Gilmo Yang)
## BIO
----------
Undergraduate Student majoring Computer Science & Engineering, Interested in Cognitive Architecture, Cellular Automata, and other DL, ML branches of study.

## Organization
----------
Seoul National University, Dept. of Computer Science & Engineering

AttentionX

## Contact
----------
[E-mail](atlas.yang3598@gmail.com)

[GitHub](https://github.com/AtlasYang)

[Blog](https://blog.naver.com/epsilon2718)



-----------------
You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

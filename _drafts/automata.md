# finite Automata
* (rough하게 말하자면) state들의 유한 집합, 주어진 규칙에 따라 하나의 state에서 다른 state로 transition(상태 전이)한다.
* 표현방법: 그래프로!

# 기본 개념
* Alphabet(Σ): Finite set of symbols.
* String(Σ*): The set of strings over an alphabet Σ is the set of lists, each element of which is a member of Σ.
* Language: a subset of string Σ*.

# DFA
* 구성요소
  * Q: finite set of states
  * Σ: input alphabet
  * δ: transition function from Q X Σ to Q.
  * q_0 in Q: start state
  * F: a set of acceptin, final states. It is a subset of Q.

# Language
모든 종류의 automata는 language를 정의한다. <br>

**def) L(A)** If A is an automata, then the set of strings L(A) is language of A. For all w ∈ A, δ(q_0, w) ∈ F.

**def) Regular Language** 언어 L을 accept하는 DFA가 존재하고, 이 DFA가 오직 L에 있는 문자열만 accept한다면 L은 Regular Language이다.

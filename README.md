# FSM Template Documentation

## 소개
FSM(Finite State Machine) 템플릿은 다양한 상태를 효율적으로 관리하고 전환할 수 있는 구조를 제공하는 유틸리티입니다. 이 템플릿은 Unity와 같은 게임 엔진에서 상태 기반 로직을 구현하려는 개발자를 위해 설계되었습니다.

---

## 주요 구성 요소

### 1. **Fsm 클래스**
`Fsm` 클래스는 상태를 관리하고 전환을 처리하는 핵심 클래스입니다.

- **상태 관리**: 상태를 추가하거나 제거 가능.
- **상태 전환**: 현재 상태에서 다음 상태로 전환하는 로직 포함.
- **업데이트 루프**: 현재 상태를 지속적으로 업데이트.

**코드 예제:**
```c#
using UnityEngine;
using System.Collections.Generic;

public class Fsm<T>
{
    protected SortedList<T, FsmState<T>> stateList = new SortedList<T, FsmState<T>>();
    protected FsmState<T> curState = null;
    protected FsmState<T> nextState = null;

    public virtual void Clear()
    {
        stateList.Clear();
        curState = null;
        nextState = null;
    }

    public virtual void AddFsm(FsmState<T> _state)
    {
        if (stateList.ContainsKey(_state.getState))
            return;
        stateList.Add(_state.getState, _state);
    }

    public virtual void SetState(T _state)
    {
        if (!stateList.ContainsKey(_state))
            return;
        nextState = stateList[_state];
    }

    public virtual void Update()
    {
        if (nextState != null)
        {
            curState?.End();
            curState = nextState;
            nextState = null;
            curState.Enter();
        }

        curState?.Update();
    }
}
```
---
### 2. **FsmState 클래스**
`FsmState` 클래스는 각 상태의 동작을 정의하기 위한 추상 클래스입니다. 상태의 진입, 업데이트, 종료 이벤트를 오버라이드할 수 있습니다.

**코드 예제:**
```c#
using UnityEngine;

public abstract class FsmState<T>
{
    private T state;

    public FsmState(T _state)
    {
        state = _state;
    }

    public T getState
    {
        get { return state; }
    }

    public virtual void Enter() { }
    public virtual void Update() { }
    public virtual void End() { }
}
```

---
### 3. FsmFactory 클래스
`FsmFactory` 클래스는 FSM 인스턴스를 생성하고 상태를 추가하며 초기 상태를 설정하는 유틸리티를 제공합니다.

**코드 예제:**
```c#
using System;

public static class FsmFactory
{
    public static Fsm<T> CreateFsm<T>()
    {
        return new Fsm<T>();
    }

    public static void AddState<T>(Fsm<T> fsm, FsmState<T> state)
    {
        if (fsm == null || state == null)
            throw new ArgumentNullException("FSM or state cannot be null.");
        fsm.AddFsm(state);
    }

    public static void SetInitialState<T>(Fsm<T> fsm, T initialState)
    {
        if (fsm == null)
            throw new ArgumentNullException("FSM cannot be null.");
        fsm.SetState(initialState);
    }
}
```
---
## 사용 방법
---
1. FSM 생성
```c#
Fsm<string> fsm = FsmFactory.CreateFsm<string>();
```
2. 상태 추가
```c#
fsm.AddFsm(new ExampleState("Idle"));
```
5. 초기 상태 설정
```c#
FsmFactory.SetInitialState(fsm, "Idle");
```
6. 업데이트 호출
```c#
fsm.Update();
```

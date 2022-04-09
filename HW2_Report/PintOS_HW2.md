# EC4205 운영체제 프로젝트 과제: PintOS HW2

작성자: 팀1(20185054 김현서, 20195023 김민석)
저장소: [dodok8/GIST_PintOS](https://github.com/dodok8/GIST_PintOS)

## Problem 1

현재의 PintOS는 `ready_list`에 들어 온 것을 순차적으로 실행한다. 이를 더 중요한(더 높은 우선 순위를 가진) 쓰레드부터 실행 될 수 있도록 다음 2가지를 구현한다.

- `ready_list`에서 가장 높은 우선 순위를 가진 쓰레드가 다음에 실행되도록 한다.
- `ready_list`에 추가된 쓰레드가 현재 쓰레드 보다 우선 순위가 높으면, 새 쓰레드가 실행되도록 한다.

### Algorithm

 1. 새로 생긴 thread가 현재 실행중인 thread보다 priority가 높은 경우 switching
 2. 현재 실행중인 thread의 priority가 바뀌었고, 다른 `ready_list`의 thread보다 priority보다 낮아졌을 때 switching

이를 위해, read_list에 추가되는 경우, priority에 따라 높은 우선 순위가 먼저 들어가도록 하였고, `thread_create`와 `thread_set_priority`에서 switching이 필요한 경우, `thread_yield`를 호출하였다.

### Implementation

```c
// function to compare threads and use result in sort (pintos 2nd project)
bool less_pri_comp(struct list_elem *a, struct list_elem *b, void *aux)
{
  struct thread *a_thread = list_entry(a, struct thread, elem);
  struct thread *b_thread = list_entry(b, struct thread, elem);

  if (a_thread->priority > b_thread->priority)
    return true;
  else
    return false;
}
```

먼저 우선 순위를 비교하기 위해 `less_pri_comp`를 만들었다.

```c
void thread_yield(void)
{
  ...
  if (cur != idle_thread)
  {
    list_insert_ordered(&ready_list, &cur->elem, &less_pri_comp, NULL);
  }
  ...
}

```

```c
void thread_unblock(struct thread *t)
{
  ...
  list_insert_ordered(&ready_list, &t->elem, &less_pri_comp, NULL);
  ...
}
```

이렇게 스케쥴링을 위해 `ready_list`에 접근 할 때, 우선 순위에 따라 정렬되게 했으므로, `ready_list`는 항상 정렬된 상태로 있다.

```c
void thread_set_priority(int new_priority)
{
  ASSERT(PRI_MIN <= new_priority && new_priority <= PRI_MAX); // priority로 적합하지 않은 값이 들어올 경우 정지
  ...
  struct thread *next_t = list_entry(list_begin(&ready_list), struct thread, elem);
  
  if (!list_empty(&t->donation_list)) // Donation 처리를 위한 코드, 다음 단락에서 더 자세히 설명
  {
    ...
  }
  
  if(t->priority < next_t->priority)
    thread_yield();
}
```

만약 현재 쓰레드의 우선 순위가 변경된 경우, `read_list`의 다른 요소들은 이미 정렬된 상태이므로, 현재 쓰레드와 `ready_list`의 바로 다음 쓰레드를 비교해서 `thread_yield`를 호출하는 것으로 switching을 할 수 있다.

## Problem 2

### Algorithm

### Implementation

## Result

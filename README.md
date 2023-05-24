# 가비지 컬렉션(Garbage-Collection)
### 가비지 컬렉션이란 ?
* 프로그램을 개발 하다 보면 유효하지 않은 메모리인 가비지(Garbage)가 발생하게 되는데 이러한 가비지는 불필요하게 메모리를 점유하게 됩니다. 
* 가비지 컬렉션은 메모리 관리 기법 중 하나로 프로그램이 동적으로 할당했던 메모리 영역 중에서 필요없게 된 영역을 해제하는 기능입니다.

### 가비지 컬렉션의 필요성
* 현대적인 언어가 아닌 과거 언어인 경우, 메모리 관리를 직접 해줘야 하는 언어들은 크게 두 가지 문제점을 가지고 있습니다.
1. 필요 없는 메모리를 비우지 않았을 때 : 메모리 사용을 마쳤을 때 비우지 않을 경우 메모리 누수가 발생할 수 있고 장기적인 관점에서 심각한 문제가 발생할 수 있다.
2. 사용중인 메모리 비우기 : 존재하지 않는 메모리에 접근하려고 하면 프로그램이 중단되거나 메모리 데이터 값이 손상될 수 있다.

가비지 컬렉션은 이러한 문제를 해결하기 위해 프로그램이 실행되는 동안 더 이상 사용되지 않는 메모리를 식별하고 자동으로 해제하여 메모리 누수를 방지합니다.

# Python의 메모리 할당
* Pytho의 메모리에는 크게 4개 구조가 있습니다.
  * 코드 영역 : 프로그램 실행할 코드를 저장
  * 데이터 영역 : 전역변수 등을 저장
  * 힙 영역(heap) : 동적 할당된 변수나 메소드 저장
  * 스택 영역 (stack) : 지역변수 등을 저장

아래의 코드를 예로 들면,

        def f2(x):
        x = x+1
        return x 

        def f1(x):
        x = x*2
        y = f2(x)
        return y

        #main 
        y = 5 
        z = f1(y)

        print(z)
        >>> 11

* main에서 y=5를 선언했다. 5는 heap영역에, 변수 y는 stack 영역에 저장됩니다.
* 이어서 선언한 z와 f1() 는 stack영역에 추가되고, f1에 담은 y는 5를 참조합니다. 5는 다시 f1(x)의 x에 전달됩니다.
* f1(5) 가 실행되면, x=x*2 에 의해 10은 heap영역에 저장되고 x 는 10을 가리킵니다.
* y=f2(x)가 실행되면, f2()는 stack영역에 저장되고 10을 가리킵니다.
* x=x+1로 인해 11이 heap영역에 추가되고, x는 11을 가리키게 됩니다.
* return x로 인해 11이 f1(x)의 y에 담기고, return y로 인해 z에 11이 담깁니다.
* f1()과 f2()는 stack영역에서 사라집니다.

# python 가비지 컬렉션 메소드
* python에서의 가비지 컬렉션 메소드에는 대표적으로 2가지가 있습니다.

1. Reference count
2. Generational Garbage Collection

## 1. Reference count(참조 횟수 계산)
* Pytho의 주된 가비지 콜렉션 메커니즘은 Reference count입니다. Pytho에서 어떤 객체를 생성하면, C  수준 객체는 이 Pytho 객체의 타입과 reference count를 가지고 있게 됩니다.
  * reference count는 객체가 참조될 때마다 +1, 해제될 때 -1로 계산됩니다. 이렇게 각 객체의 참조 횟수를 count하여 0이 되면 해당 객체를 해제하는 방식을 말합니다.
  * 이때 객체의 카운트가 0이라는 뜻은 더이상 이 객체에 접근하고 있는 코드가 없다는 의미이므로 메모리에서 삭제, 즉 할당 해제(deallocation)를 할 수 있게 되고, 바로 이 때 객체가 지워지게 됩니다. 따라서 안전하게 해당 메모리를 "즉시" 확보 가능합니다.

#### Reference count 확인
* sys 라이브러리를 사용해서 특정 객체의 Reference count를 확인할 수 있으며, 참조 횟수를 증가시키는 방법은 다음과 같습니다.
    * 객체를 변수에 할당
    * 객체를 리스트, 튜플과 같은 자료구조에 하거나 인스턴스 프로퍼티로 추가
    * 함수에 파라미터로 전달
 
위의 내용을 테스트 해보면, 

        >>> import sys
        >>> a = 'hello'
        >>> sys.getrefcount(a)
        2

Reference count가 2인걸 알 수 있습니다. 1은 처음 객체가 a 에 할당되는 순간에 증가하고, 2는 sys.getrefcount()함수에 전달되는 순간에 증가하게 됩니다.

재시작하고 리스트와 딕셔너리에 객체를 추가해보면

        >>> import sys
        >>> a = 'hello'
        >>> sys.getrefcount(a)
        2
        >>> b = [a]
        >>> sys.getrefcount(a)
        3
        >>> c = {'first': a}
        >>> sys.getrefcount(a)
        4

 list나 dictionary에 추가될 때 a의 참조 횟수가 증가하는 것을 알 수 있습니다.
 
하지만 객체가 자기 자신을 가르키는 '순환참조'를 하게 될 경우 Reference count 방식으로는 메모리에서 해제 될 수 없습니다.
      
      >>> a = []
      >>> a.append(a)
      >>> del a
      
위의 코드와 같이 a의 참조 횟수는 1이지만 이 객체는 더 이상 접근할 수 없으며 Reference count 방식으로는 메모리에서 해제될 수 없는 것을 볼 수 있습니다.      

또 다른 예로는 서로를 참조하는 객체입니다.

    >>> a = Func_pr() # 0x01
    >>> b = Func_pr() # 0x02
    >>> a.x = b # 0x01의 x는 0x02를 가리킨다.
    >>> b.x = a # 0x02의 x는 0x01를 가리킨다.
    # 0x01의 레퍼런스 카운트는 a와 b.x로 2다.
    # 0x02의 레퍼런스 카운트는 b와 a.x로 2다.
    >>> del a # 0x01은 1로 감소한다. 0x02는 b와 0x01.x로 2다.
    >>> del b # 0x02는 1로 감소한다.

마지막 상태에서 0x01.x와 0x02.x가 서로를 참조하고 있기 때문에 레퍼런스 카운트는 둘 다 1이지만 0에 도달할 수 없는 garbage(쓰레기)가 되는 것을 알 수 있습니다.

이러한 유형의 문제를 reference cycle(참조 주기)이라고 하며 reference counting으로 해결할 수 없습니다.

## 2. Generational Garbage Collection(세대 단위 쓰레기 수집)
* Generational Garbage Collection에는 두가지 핵심이 있습니다.

1. 세대(generation)
* 가비지 컬렉터는 메모리의 모든 객체를 추적합니다.
* 새로운 객체는 1세대 가비지 수집기에서 life(수명)를 시작합니다.
* Python이 세대에서 가비지 수집 프로세스를 실행하고 객체가 살아남으면, 두 번째 이전 세대로 올라갑니다.
* Python 가비지 수집기는 총 3세대이며, 객체는 현재 세대의 가비지 수집 프로세스에서 살아남을 대마다 이전 세대로 이동합니다.

2. 임계(threshold)값
* 각 세대마다 가비지 컬렉터 모듈에는 임계값 개수의 개체가 있습니다.
* 객체 수가 해당 임계값을 초과하면 가비지 콜렉션이 콜렉션 프로세스를 trigger(추적) 합니다.
* 해상 프로세스에서 살아남은 객체는 이전 세대로 옮겨집니다.

가비지 컬렉터는 내부적으로 generation(세대)과 threshold(임계값)로 가비지 컬렉션 주기와 객체를 관리합니다. 세대는 0~2세대로 구분되고 최근 생성된 객체는 0세대(young)에 들어가고 오래된 객체일수록 2세대(old)로 이동합니다. 당연히 한 객체는 단 하나의 세대에만 속합니다. 

Generational Garbage Collection은 GC module의 get_threshold() method를 사용하여 구성된 임계값을 확인할 수 있습니다.

    >>> import gc
    >>> gc.get_threshold()
    (700, 10, 10)

각각 threshold 0, threshold 1, threshold 2를 의미하는데 n세대에 객체를 할당한 횟수가 threshold n을 초과하면 가비지 컬렉션이 수행됩니다.

또한 get_count() method를 사용하여 각 세대의 객체 수를 확인할 수 있습니다.

    >>> import gc
    >>> gc.get_count()
    (121, 9, 2)

(위 값은 method를 호출할 때마다 변경됩니다.) 위 코드에서는 youngest generation(가장 어린 세대)에 121개의 객체, 다음 세대에 9개의 객체 oldest generation(가장 오래된 세대)에 2개의 객체가 있는 것을 알 수 있습니다.

gc module에서 set_threshold() method를 사용하여 가비지 컬렉션 트리거 임계값을 변경할 수 있습니다.

    >>> import gc
    >>> gc.get_threshold()
    (700, 10, 10)
    >>> gc.set_threshold
    (1000, 15, 15)
    >>> gc.get_threshold()
    (1000, 15, 15)

임계 값을 증가시키면 가비지 컬렉션이 실행되는 빈도가 줄어들며 죽은 객체를 오래 유지하는 cost(비용)로 프로그램에서 계산 비용이 줄어듭니다.

# Pytho의 메모리 관리 요약
* Pytho은 동적 메모리 할당을 기본으로 합니다.
* 즉, 프로그래머가 직접 관리하는게 아니라 reference count와 garbage collection으로 메모리 관리가 된다는 뜻입니다.
  * 객체가 생성되면, Pytho은 객체를 메모리와 세대 0에 assign합니다.
  * 임계치를 오버하는 경우, collect_generations()를 실행한다. 2세대부터 역순으로 검사합니다.
  * 객체 수가 임계치를 넘으면 gc.collect()를 실행해 garbage collection process를 수행합니다.
  * gc.collect()는 unreachable 객체의 개수를 반환한다. 도달할 수 없는 객체란, 더이상 사용되지 않는 객체를 말한다. 이들은 메모리에서 해제됩니다.
  * reachable 객체는 다음 세대로 이동됩니다.

# 예제 코드
* 순환 참조 (Circular reference)


      class A:
      def __init__(self):
      self.b = B(self)

      class B:
      def __init__(self, a):
      self.a = a

      a = A()
      b = a.b
      del a
      del b

위 예제에서는 A 객체와 B 객체가 서로를 참조하고 있습니다.
이 경우에 가비지 컬렉션은 참조되지 않는 객체로 판단하지 못하고 계속해서 메모리에 남아있게 됩니다.

순환 참조(Circular reference) 문제를 해결하기 위해 약한 참조(Weak reference)를 사용합니다. Python의 weakref 모듈을 활용하여 순환 참조가 발생하지 않도록 조치할 수 있습니다.


    import weakref

    class A:
        def __init__(self):
          self.b = weakref.ref(B(self))

    class B:
        def __init__(self, a):
          self.a = a

    a = A()
    b = a.b()
    del a
    del b

* 전역 변수 (Global variables)

      my_list = []

      def add_to_list(item):
      my_list.append(item)

위 예제에서는 my_list가 전역 변수로 선언되었고, 함수 add_to_list를 통해 요소가 추가됩니다.
하지만 리스트가 계속해서 커지면서 메모리를 계속해서 차지하게 되고, 이는 메모리 누수로 이어질 수 있습니다.

이러한 문제를 해결하기 위해서는 전역 변수(Global variables)를 최소화하고, 필요한 경우에는 클래스나 함수 내에 지역 변수(Local variables)로 사용합니다. 지역 변수는 해당 함수나 클래스의 실행이 완료될 때 함께 사라지므로 메모리 누수를 방지할 수 있습니다.


    def process_data(data):
    local_list = []
    local_list.append(data)
    # 데이터 처리 로직


위의 코드와 같이 전역 변수 대신 지역 변수인 local_list를 사용하여 메모리 누수를 방지합니다.

------
###### 참고사이트 : https://medium.com/dmsfordsm/garbage-collection-in-python-777916fd3189, https://velog.io/@mquat/OS-Garbage-collector-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%ED%8C%8C%EC%9D%B4%EC%8D%AC%EC%9D%98-Memory-%EC%82%AC%EC%9A%A9, https://openai.com/product/gpt-4

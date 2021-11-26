# LiveData
## 1. Observer
 ```Kotlin
class MainActivity : AppCompatActivity() {
 
    override fun onCreate(savedInstanceState: Bundle?) {
        // activity 설정 관련 소스 생략
        
        DB에서 초기 아이템 목록 호출
        UI 업데이트
        
        추가 버튼 클릭 리스너 {
            아이템 추가
            UI 업데이트
        }
        
        삭제 버튼 클릭 리스너 {
            아이템 삭제
            UI 업데이트
        }
    }
}
```
예전에 LiveData의 존재를 모르고 마구잡이로 개발을 했을 때 나는 위와 같은 코드를 작성했었다.

메모를 저장하는 앱이였는데 메모를 불러오거나, 추가, 삭제할 때마다 매번 일일이 UI를 업데이트해줘야 한다는 문제가 있었다.
 ```Kotlin
        삭제 버튼 클릭 리스너 {
            아이템 삭제 // DB에서 아이템을 삭제해야하는데 이게 너무 오래걸렸음
            UI 업데이트 // 그래서 삭제가 되기도 전에 업데이트가 이루어져버림
        }
```
더군다나 DB에서 아이템을 삭제하는 경우에는

삭제 후 UI 업데이트가 이루어져야하는데

삭제 작업은 메인 스레드에서 이루어지지 않는 데다가 오래 걸리는 작업이기 때문에

UI 업데이트가 이루어진 후 아이템 삭제가 돼서 실제 화면에서는 삭제가 되지 않는 것처럼 보이는 문제가 생겼다.

<img src="https://user-images.githubusercontent.com/48902047/143533254-78200ea5-5aaf-4e05-baa4-30867431c089.png"></img>

아이템 추가나 삭제 같은 데이터 변경을 요청했을 때

누군가가 데이터의 변경이 완료되기를 기다리고 있다가 나에게 알려주면 좋겠다는 생각을 했다.

<img src="https://user-images.githubusercontent.com/48902047/143533311-5cbb00d6-0a7b-4e2c-b1e6-07e0edf318f5.png"></img>

중학생 때 친구들끼리 동전을 걸고 하는 판치기를 가끔 하곤 했었다.

선생님께 걸리면 혼나기 때문에 꼭 한 명은 교실 뒷문에서 망을 보는 역할을 해야 했다.

선생님이 오시는지 감시를 하고 있다가 애들한테 알려주는 **관찰자 역할**

그것이 바로 이번 목차에서 설명하고자 했던 Observer이다. (이야 이거 설명하려고 진짜 먼길 돌아왔다)

안드로이드에서의 Observer는 데이터가 변경되는지 감시하고 있다가 UI 컨트롤러 (Activity)에게 알려준다.

알림을 들은 UI 컨트롤러는 그 데이터를 가지고 UI를 업데이트하는 그런 개념인 것이다.

단 여기서 Observer는 아무 데이터나 감시할 수 있는 게 아니라

LiveData라는 데이터 홀더 클래스가 가지고 있는 데이터만 감시할 수 있다.

이어서 LiveData에 대해 알아보자.

## 2. LiveData
 ```Kotlin
var height = 183
```
만약 키를 저장하는 변수 height의 값이 변경되는지 감시하고 싶다고 치자.

  ```Kotlin
var height = MutableLiveData<Int>() // int형 값을 넣을 거라는 걸 명시해주어야 한다.
 
height.value = 183
```

위와 같이 MutableLiveData 객체를 생성하고 값을 넣을 때는. value를 통해 넣어주면 된다.

이제부터 이 데이터는 Observer가 감시할 수 있는 데이터가 되는 것이다.

<img src="https://user-images.githubusercontent.com/48902047/143534074-893caade-d261-45cf-9718-27f37a4a0089.png"></img>

 ```Kotlin
public class MutableLiveData<T> extends LiveData<T> {
 
    public MutableLiveData(T value) {
        super(value);
    }
 
    public MutableLiveData() {
        super();
    }
 
    @Override
    public void postValue(T value) {
        super.postValue(value);
    }
 
    @Override
    public void setValue(T value) {
        super.setValue(value);
    }
}
```
MutableLiveData 클래스는 LiveData를 상속받는 클래스이다.

LiveData에 postValue와 setValue (값을 넣는 메서드)를 추가한 클래스라고 할 수 있다.

그렇다. MutableLiveData는 말 그대로 값의 수정이 가능하고

LiveData는 값의 수정이 불가능하다.

<img src="https://user-images.githubusercontent.com/48902047/143534295-b92e01ba-af5b-4801-b4be-1daaf57de518.png"></img>

분명 LiveData를 사용하면 값의 변화를 감지할 수 있다 했는데..

값의 수정이 불가능하면.. 애초에 값이 변화하질 않잖아..?

만약 클린 아키텍처에 대해 알고 있다면 그 이유를 쉽게 파악할 수 있다.

예를 들어 아키텍처 패턴 중 하나인 MVVM 패턴을 사용한다고 하면 아래와 같은 구조를 가지게 될 것이다.

<img src="https://user-images.githubusercontent.com/48902047/143534612-5d461092-6a4d-46a3-be6c-32d87f71c457.png"></img>

1. 웹에서 데이터를 가져오는 등의 값 변경이 일어난다.

2. MutableLiveData와 LiveData가 연결되어 있다.

3. 관찰자는 MutableLiveData가 아닌 LiveData를 관찰한다.

즉 데이터 변경이 일어나면 1-2-3 순서를 거쳐 관찰자가 변경되었다는 사실을 알게 되고 UI를 업데이트한다.

2 를 제외하고 1에서 3으로 바로 가도 되는데 굳이 이렇게 하는 이유는

**UI 컨트롤러(액티비티, 프래그먼트)가 값을 직접 수정하지 못하게 하기 위함이다.**

## 3. LiveData의 장점

+ 데이터와 UI 상태 일치 보장
  + LiveData는 데이터가 변경될 때마다 Observer 객체에게 알려주고, Observer는 알림을 받을 때마다 대신 UI를 업데이트하므로 데이터와 UI 상태 일치를 보장한다.
+ 메모리 누수 없음
  + Observer는 Activity나 Fragment의 수명 주기를 따르며 수명 주기가 끝나면 자동으로 삭제된다. 따로 메모리를 해제하거나 하는 작업을 하지 않아도 된다는 뜻이다.
+ 중지된 활동으로 인한 비정상 종료 없음
  + Activity나 Fragment가 백 스택에 있을 때 Observer는 비활성 상태가 되며, 이때는 어떤 LiveData 이벤트도 수신받지 않는다.
+ 수명 주기를 수동으로 처리하지 않음
  + UI 구성요소는 데이터를 관찰할 수만 있고 관찰을 중지하거나 다시 시작하지 않는다. 대신 수명 주기 상태의 변경을 인식하기 때문에 이를 통해 자동으로 관리한다.
+ 최신 데이터 유지
  + 수명 주기가 비활성 -> 활성으로 다시 돌아올 때 최신 데이터를 수신한다.
+ 적절한 구성 변경
  + 기존 방식은 기기를 회전하기 전 saveInstanceState를 이용해 기존 데이터를 보관해두었다가 회전 후 데이터를 가져와 다시 복원하는 방식이었으나 LiveData를 사용하면 이런 작업을 하지 않아도 최신 데이터를 즉시 받게 된다.
+ 리소스 공유
  + LiveData 객체가 시스템 서비스에 한 번 연결되면 LiveData가 필요한 모든 곳에서(모든 Observer가) LiveData 객체를 관찰할 수 있다.

## 4. 더 알아보기
### LifeCycleOwner

위에서 설명했듯이 Observer는 UI 컨트롤러의 생명주기를 따른다.

Observer가 어떤 액티비티 혹은 어떤 프래그먼트의 수명주기를 따를지 정해주어야 한다.

액티비티 같은 경우는 this 키워드를 이용해 현재 액티비티를 지정해주면 되는데

**프래그먼트 같은 경우**는 this를 사용하면 안 된다.

그 [이유](https://pluu.github.io/blog/android/2020/01/25/android-fragment-lifecycle/)에 대해서 잘 정리해놓은 블로그가 있어서 꼭 한 번씩 읽어보길 추천한다. 👍

3줄 요약해보자면

1. 기존의 프래그먼트 생명주기를 사용하면 복수의 Observer가 호출될 가능성이 있다.

2. 구글이 실수한 부분이고, 이를 개선하기 위해 새로운 프래그먼트 생명주기가 도입되었다.

3. this 대신에 **viewLifecycleOwner**를 사용하면 된다.

### observeForever
위에서 언급한 것처럼 Observer는 UI 컨트롤러의 생명주기를 따른다.

액티비티가 실행되면 관찰자도 감시를 시작하며 알림을 받을 수 있는 상태가 되고

액티비티가 정지되면 관찰자도 감시를 중단하며 알림을 받을 수 없는 상태가 된다.

그런데 **lifeCycleOwner와 상관없이 항상 알림을 받을 수 있는 방법이 있는데 그게 바로 observerForever 메서드**이다.

이 메서드를 사용하면 lifeCycleOwner가 없어도 관찰자를 생성하고 LiveData와 연결할 수 있으며

UI 컨트롤러의 생명주기와 상관없이 항상 알림을 받을 수 있는 상태가 된다.

 ```Kotlin
        // 일반적인 Observer 생성
        model.getAll().observe(this, Observer{ notice ->
 
        })
 
        // observerForever를 통한 생성 
        model.getAll().observeForever(Observer{ notice ->
 
        })
```

이렇게 observe대신 observerForever를 사용하면 되고 this를 넣어주지 않아도 된다.

대신 관찰자를 삭제할 때는 removeObserver 메서드를 사용하여 직접 삭제해주어야 한다.

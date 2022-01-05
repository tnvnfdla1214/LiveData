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
예전에 LiveData의 존재를 모르고 마구잡이로 개발을 했을 때 나는 위와 같은 코드를 작성했었습니다.

메모를 저장하는 앱이였는데 메모를 불러오거나, 추가, 삭제할 때마다 매번 일일이 UI를 업데이트해줘야 한다는 문제가 있었습니다.
 ```Kotlin
        삭제 버튼 클릭 리스너 {
            아이템 삭제 // DB에서 아이템을 삭제해야하는데 이게 너무 오래걸렸음
            UI 업데이트 // 그래서 삭제가 되기도 전에 업데이트가 이루어져버림
        }
```
더군다나 DB에서 아이템을 삭제하는 경우에는

삭제 후 UI 업데이트가 이루어져야하는데

삭제 작업은 메인 스레드에서 이루어지지 않는 데다가 오래 걸리는 작업이기 때문에

UI 업데이트가 이루어진 후 아이템 삭제가 돼서 실제 화면에서는 삭제가 되지 않는 것처럼 보이는 문제가 생겼습니다.

<img src="https://user-images.githubusercontent.com/48902047/143533254-78200ea5-5aaf-4e05-baa4-30867431c089.png"></img>

아이템 추가나 삭제 같은 데이터 변경을 요청했을 때

누군가가 데이터의 변경이 완료되기를 기다리고 있다가 나에게 알려주면 좋겠다는 생각을 했습니다.

<img src="https://user-images.githubusercontent.com/48902047/143533311-5cbb00d6-0a7b-4e2c-b1e6-07e0edf318f5.png"></img>

중학생 때 친구들끼리 동전을 걸고 하는 판치기를 가끔 하곤 했었습니다.

선생님께 걸리면 혼나기 때문에 꼭 한 명은 교실 뒷문에서 망을 보는 역할을 해야 했습니다.

선생님이 오시는지 감시를 하고 있다가 애들한테 알려주는 **관찰자 역할**

그것이 바로 이번 목차에서 설명하고자 했던 Observer입니다. (이야 이거 설명하려고 진짜 먼길 돌아왔다)

안드로이드에서의 Observer는 데이터가 변경되는지 감시하고 있다가 UI 컨트롤러 (Activity)에게 알려줍니다.

알림을 들은 UI 컨트롤러는 그 데이터를 가지고 UI를 업데이트하는 그런 개념인 것입니다.

단 여기서 Observer는 아무 데이터나 감시할 수 있는 게 아니라

LiveData라는 데이터 홀더 클래스가 가지고 있는 데이터만 감시할 수 있습니다.

이어서 LiveData에 대해 알아봅시다.

## 2. LiveData
 ```Kotlin
var height = 183
```
만약 키를 저장하는 변수 height의 값이 변경되는지 감시하고 싶다고 칩시다.

  ```Kotlin
var height = MutableLiveData<Int>() // int형 값을 넣을 거라는 걸 명시해주어야 합니다.
 
height.value = 183
```

위와 같이 MutableLiveData 객체를 생성하고 값을 넣을 때는. value를 통해 넣어주면 됩니다.

이제부터 이 데이터는 Observer가 감시할 수 있는 데이터가 되는 것입니다.

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
MutableLiveData 클래스는 LiveData를 상속받는 클래스입니다.

LiveData에 postValue와 setValue (값을 넣는 메서드)를 추가한 클래스라고 할 수 있습니다.

그렇다. MutableLiveData는 말 그대로 값의 수정이 가능하고

LiveData는 값의 수정이 불가능합니다.

<img src="https://user-images.githubusercontent.com/48902047/143534295-b92e01ba-af5b-4801-b4be-1daaf57de518.png"></img>

분명 LiveData를 사용하면 값의 변화를 감지할 수 있다 했는데..

값의 수정이 불가능하면.. 애초에 값이 변화하질 않잖아..?

만약 클린 아키텍처에 대해 알고 있다면 그 이유를 쉽게 파악할 수 있습니다.

예를 들어 아키텍처 패턴 중 하나인 MVVM 패턴을 사용한다고 하면 아래와 같은 구조를 가지게 될 것입니다.

<img src="https://user-images.githubusercontent.com/48902047/143534612-5d461092-6a4d-46a3-be6c-32d87f71c457.png"></img>

1. 웹에서 데이터를 가져오는 등의 값 변경이 일어난다.

2. MutableLiveData와 LiveData가 연결되어 있다.

3. 관찰자는 MutableLiveData가 아닌 LiveData를 관찰한다.

즉 데이터 변경이 일어나면 1-2-3 순서를 거쳐 관찰자가 변경되었다는 사실을 알게 되고 UI를 업데이트합니다.

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

위에서 설명했듯이 Observer는 UI 컨트롤러의 생명주기를 따릅니다.

Observer가 어떤 액티비티 혹은 어떤 프래그먼트의 수명주기를 따를지 정해주어야 합니다.

액티비티 같은 경우는 this 키워드를 이용해 현재 액티비티를 지정해주면 되는데

**프래그먼트 같은 경우**는 this를 사용하면 안 됩니다.

그 [이유](https://pluu.github.io/blog/android/2020/01/25/android-fragment-lifecycle/)에 대해서 잘 정리해놓은 블로그가 있어서 꼭 한 번씩 읽어보길 추천한다. 👍

3줄 요약해보자면

1. 기존의 프래그먼트 생명주기를 사용하면 복수의 Observer가 호출될 가능성이 있다.

2. 구글이 실수한 부분이고, 이를 개선하기 위해 새로운 프래그먼트 생명주기가 도입되었다.

3. this 대신에 **viewLifecycleOwner**를 사용하면 된다.

### observeForever
위에서 언급한 것처럼 Observer는 UI 컨트롤러의 생명주기를 따릅니다.

액티비티가 실행되면 관찰자도 감시를 시작하며 알림을 받을 수 있는 상태가 되고

액티비티가 정지되면 관찰자도 감시를 중단하며 알림을 받을 수 없는 상태가 됩니다.

그런데 **lifeCycleOwner와 상관없이 항상 알림을 받을 수 있는 방법이 있는데 그게 바로 observerForever 메서드**입니다.

이 메서드를 사용하면 lifeCycleOwner가 없어도 관찰자를 생성하고 LiveData와 연결할 수 있으며

UI 컨트롤러의 생명주기와 상관없이 항상 알림을 받을 수 있는 상태가 됩니다.

 ```Kotlin
        // 일반적인 Observer 생성
        model.getAll().observe(this, Observer{ notice ->
 
        })
 
        // observerForever를 통한 생성 
        model.getAll().observeForever(Observer{ notice ->
 
        })
```

이렇게 observe대신 observerForever를 사용하면 되고 this를 넣어주지 않아도 됩니다.

대신 관찰자를 삭제할 때는 removeObserver 메서드를 사용하여 직접 삭제해주어야 합니다.

## 예제
### LiveData를 이용해 Button을 누르면 TextView의 숫자를 1씩 증가시키기

 ```Kotlin
 //activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/text_test"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@id/btn_change"
        app:layout_constraintVertical_chainStyle="packed"/>

    <Button
        android:id="@+id/btn_change"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="ADD 1"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/text_test"
        app:layout_constraintBottom_toBottomOf="parent"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```
텍스트 뷰랑 버튼하나있어서 버튼누르면 텍스트뷰 변하는것입니다.
 ```Kotlin
 //MainActivity.kt
package com.imaec.livedataex

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.Observer
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {

    private var liveText: MutableLiveData<String> = MutableLiveData()
    private var count = 0 // button을 누르면 증가 될 숫자

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // LiveData의 value의 변경을 감지하고 호출
        liveText.observe(this, Observer {
            // it로 넘어오는 param은 LiveData의 value
            text_test.text = it
        })

        btn_change.setOnClickListener {
          // liveText의 value를 변경
            // liveText 자체를 변경시키면 안됨
            liveText.value = "Hello World! ${++count}"
        }
    }
}
```
버튼을 누르면 livedata에 value값을 변경해주고 그러면 observer가 이벤트를 생성할테니 그 이벤트에

textview의 text를 바꿔주는것을 담은것입니다.

## live data에 초기에 값을 지정해주는 방법을 알아보자
LiveData의 값을 초기에 지정해주는 방법은 여러가지가 있는데 두 가지만 알아보자면 하나는 **kotlin의 apply block을 사용하는 것**이고 하나는 **kotlin의 extension function을 사용하는 방법**이 있습니다.

### 1. apply block 이용
 ```Kotlin
private var liveText: MutableLiveData<String> = MutableLiveData<String>().apply {
    value = "Hello World! ${++count}"
}
```
이렇게 초기화할때 .apply 블록에다가 value를 넣어주는 방법이다  이렇게 사용할때는 

MutableLiveData의 생성자에 Type을 꼭 지정해주어야합니다.

### 2. extension function 사용
Set 함수 정의
 ```Kotlin
/**
 * MutableLiveData의 value를 정의 해주는 함수
 * @param value MutableLiveData의 value
 * @return MutableLiveData의 instance
 */
private fun MutableLiveData<String>.set(value: String) : MutableLiveData<String> {
    this.value = value
    return this
}
```
함수 사용
 ```Kotlin
private var liveText: MutableLiveData<String> = MutableLiveData<String>().set("Hello World! ${++count}")
```
이렇게 set을 통해서도  초기 값을 지정해줄수있습니다.

## setValue()와 postValue()
이전까지 liveData 연습하면서 getter setter 를 멍청하게 다 만들어가면서 했었습니다.
![image](https://user-images.githubusercontent.com/48902047/148242466-72790305-3b19-4e50-87a1-17c7d34a0293.png)
어쩃든 이런거 코틀린에서는 할필요 없습니다.

그래서 이런거 getter setter 설정안하고 어떻게 쓰는가?

setter로는 

setValue() 함수와 postValue() 함수가 있습니다.

둘다 mutatableLivedata의 값을 설정해주는 함수인데 살짝 차이가 있다 어느 쓰레드에서 처리하는가의 차이인데 이부분은 적절하게 설정해서 사용해야하는 것 입니다.

 

### setValue()
 setValue()는 메인 쓰레드에서 LiveData의 값을 변경해줍니다. 메인 쓰레드에서 바로 값을 변경해주기 때문에 setValue() 함수를 호출한 뒤 바로 밑에서 getValue() 함수로 값을 읽어오면 변경된 값을 가져올 수 있습니다. 중요한 점은, setValue()는 메인 쓰레드에서 값을 dispatch 하기 때문에 백그라운드에서 setValue()를 호출한다면 오류가 나게 됩니다. setValue()가 동작하지 않는다면, 해당 함수가 호출되는 쓰레드가 메인 쓰레드인지 체크해봐야 합니다.

### postValue()
 postValue()는 setValue()와 다르게 백그라운드에서 값을 변경합니다. 백그라운드 쓰레드에서 동작하다가 메인 쓰레드에 값을 post 하는 방식으로 사용됩니다. 함수 내부적으로는 아래와 같은 코드가 실행됩니다.
```Kotlin
new Handler(Looper.mainLooper()).post(() -> setValue())
```
메인 쓰레드에 적용되기 전에 postValue()가 여러 번 호출된다면 모든 값이 적용되는 것이 아니라 가장 최신의 값이 적용됩니다. 따라서 postValue()를 호출한 뒤 바로 getValue()로 값을 읽으려고 한다면 변경된 값을 읽어오지 못할 가능성이 높습니다. Hander()를 통해 메인 쓰레드에 값이 전달되기 전에 getValue()를 호출하기 때문입니다. LiveData의 값을 즉각적으로 변경해야 한다면 postValue()가 아닌 setValue()를 사용해야 합니다.

즉 비동기적으로 처리되는가 + 어느 쓰레드에서 처리하는가의 차이가있습니다.

사용은 이런식으로 
 
![image](https://user-images.githubusercontent.com/48902047/148243142-5db208cd-229b-4531-9d37-e4779431cb10.png)
 
livedata.postValue(바꿔줄값) 이런식으로 넣으면 됩니다.

getter로는 getValue()를 사용하면 됩니다.

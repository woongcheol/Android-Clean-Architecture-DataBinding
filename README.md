`최근 업데이트 : '22. 08. 10.`

![image](https://user-images.githubusercontent.com/86638578/183851710-91769369-8591-4ce4-b6d8-f057530005aa.png)
## 개요
#### 라이브러리 DataBinding은 다음 [자료](http://dev-imaec.tistory.com/37)를 참고하여 구현했으며, 지속적으로 업데이트 될 예정입니다.
#### 이외에도 [Clean Architecture Repo](https://github.com/woongcheol/Android-Clean-Architecture)에서 다양한 개념들을 확인하실 수 있습니다.

</br>

## 본론
### 1. Android DataBinding이란?

- 특정 데이터나 데이터 소스로부터 UI를 결합시켜주는 라이브러리입니다. 이는 아키텍쳐 권장사항에 해당하는 Jetpack 라이브러리의 AAC(Android Architecture Components)에서 제공됩니다.  

- 사용 목적은 특정 데이터를 화면에 보여줌에 있어 데이터가 변경됨에 따라 화면도 함께 변할 수 있게 하기 위함입니다.  


- 데이터와 UI가 연결되는 방식에서 프로그래밍적 연결과 선언적 연결로 표현되곤하는데 DataBinding은 선언적 연결로, 레이아웃 파일에서 특정 위젯에 직접 접근합니다. 이와 반대로 프로그래밍적 연결은 클래스 내에서 findViewById 메소드를 통해 특정 위젯을 찾는 것을 의미합니다. 이에 대한 차이는 후술합니다.

### 2. 기능

- 컴파일 단계에서 레이아웃 파일(XML)을 변환한 java class가 자동으로 생성되어 별도의 findViewById 지정을 하지 않아도 됩니다.

- DataBindingUtil과 같은 object를 이용해 앞서 생성된 binding class를 인플레이트하여 인스턴스화 시켜 특정 뷰에 접근할 수 있게 됩니다.

- Null Safety, 여러 설정이 존재하는 레이아웃의 경우 해당 설정에 맞는 뷰만 찾아내어 Null로부터 안전하며

- Type Safety, 레이아웃 내부에 정확한 뷰 타입을 찾아 맵핑하므로 해당 뷰의 속성에 접근할 때 이에 맞는 속성 값만 노출됩니다.

- 레이아웃 파일(XML)의 data 태그 등 내부 변수 작성으로 LivaData, DTO 클래스 등 각종 변경사항에도 UI와 데이터 바인딩이 보장됩니다.

- View와 ViewModel의 분리에 용이하여 MVVM 패턴과 함께 사용됩니다.

### 3. 고려사항

- 생성되는 레이아웃 파일마다 binding class가 계속 생성되기 때문에 앱의 용량이 커질 수 있습니다.

- 빌드의 속도가 느려져 전체적인 개발 속도에 영향을 미칠 수 있습니다.

- 레이아웃 파일에 오류가있을 경우 특정 에러를 제외하고 전혀 다른 exception을 발생하는 등의 문제를 일으키므로 데이터가 제대로 넘어가지 않는 경우 이유를 확인하기가 어렵습니다.

### 4. findViewById, ViewBinding 비교

- findViewByid는 id 확인을 위해 ViewGroup을 전부 돌면서 확인하는 로직으로 속도가 느리며, 안정성 면에서도 좋지않아 지양하는 코드가 되었습니다.

- ViewBinding은 컴파일 속도가 빠르지만 동적 UI 컨텐츠를 생성할 수 없으므로 비교적 단순 처리에 적합합니다.

## 실습
### 1. build.gradle 추가
```groovy
dataBinding {
    enabled = true
}
```   
### 2. XML 적용
```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="activity"
            type="com.example.databindingbasic.MainActivity" />
    </data>
    
</layout>
```
- 루트가 `layout`이 되었으며, 내부에 `data` 태그를 추가한다.
- `data` 태그 내에선 import 및 변수를 생성할 수 있다. 변수는 type으로 지정한 activity와 연결하여 해당 클래스에 있는 속성을 사용할 수 있다.
- import 또한 import한 해당 클래스를 참조하여 클래스의 속성을 사용할 수 있다.

### 3. 액티비티 설정
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding // 1
    var text = "No Change Text"


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main) // 2

        binding.activity = this // 3
        binding.btnChange.setOnClickListener {
            text = "Change Text" // 4
            binding.invalidateAll() // 5
        }
    }
}
```
1&#41; 레이아웃 파일명에 CamelCase로 변경, Binding이 붙은 형태의 클래스가 생성된다.
2&#41; 해당 Binding 클래스를 타입으로 하는 binding 변수를 만들고 DataBindingUtile을 통해 레이아웃을 연결해준다.
3&#41; 레이아웃 파일에서 사용한 표현식(@{}) activity에 지정했던 타입에 맞춰 값을 할당해준다.
4&#41; 레이아웃의 텍스트를 바꾸기 위에 TextView에 연결된 activity의 변수를 재할당해준다.
5&#41; 바인딩된 데이터가 변경 됐을 때 UI를 새로고침해야하며 이를 위해 invalidateAll()을 사용한다.

# Fragment

`Activity`내에서 UI 일부를 나타낸다.
+ 여러 개의 프래그먼트를 하나의 액티비티에 조합하여 창이 여러개인 UI를 만들 수 있다.
+ 하나의 프래그먼트를 여러 액티비티에서 재사용 가능하다.
+ 자체 수명주기를 가지고 자체 입력 이벤트를 받으며 액티비티 실행 중에 추가 및 제거가 가능하다.
+ 항상 액티비티 내에 포함되어 있어야 한다.
    + 호스트 액티비티의 수명주기에 직접적으로 영향을 받는다.
    + 액티비티가 일시정지 되는 경우, 그 안의 모든 프래그먼트도 일시정지 되며 액티비티가 소멸되면 모든 프래그먼트도 소멸된다.

## 사용 목적
+ 분할된 화면들을 독립적으로 구성하기 위해 사용
+ 분할된 화면들의 상태를 관리하기 위해 사용

## 생명주기 

<img src="https://user-images.githubusercontent.com/33089715/120100988-b7493380-c17e-11eb-8fbc-f9665642c909.png" width="300">

### onCreateView()
+ 프래그먼트가 자신의 UI를 처음으로 그릴 시간이 됐을 때 호출된다.
+ 프래그먼트 레이아웃의 루트이다.
    + 프래그먼트에 맞는 UI를 그리려면 메서드에서 View를 리턴해야 한다.
    + 프래그먼트가 UI를 제공하지 않는 경우 null을 반환하면 된다.
    ```kotlin
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        Log.i(tag1, "onCreateView call")
        val rootView = inflater.inflate(R.layout.fragment_chatting, container, false)
        return rootView
    }
    ```

### onActivityCreated()
+ 액티비티에서 프래그먼트를 모두 생성하고 난 다음 호출된다.
+ 액티비티의 onCreate()에서 setCreateView() 한 다음과 유사하다.
+ UI 변경 작업이 가능

### onPause()
+ 시스템이 이 메서드를 호출하는 것은 사용자가 프래그먼트를 떠난다는 첫 번째 신호이다.(프래그먼트가 소멸중이라는 의미는 아님)
+ 현재 사용자가 세션을 넘어서 지속되어야 하는 변경 사항을 적용하려면 이곳에서 해야한다.

### onDestroyView()
+ 프래그먼트에 View를 제거
+ 만약 add 할 때 backstack을 사용했다면, 다시 해당 프래그먼트로 돌아 올 때 onCreateView()가 호출된다.
+ 주의!! 프래그먼트 객체 자체는 사라지지 않고 메모리에 남아 있다.

### onDestroy()
+ 프래그먼트 객체가 파괴된다. 액티비티와의 연결은 아직 끊어지지 않았다.

### onDetach()
+ 프래그먼트 제거를 완료 하고 액티비티와의 연결도 해제시킨다.

### 호출 결과

<img width="300" alt="fragmentcall1" src="https://user-images.githubusercontent.com/33089715/120100871-093d8980-c17e-11eb-9096-a7fcd78b4ffa.png">
<img width="300" alt="fragmentcall2" src="https://user-images.githubusercontent.com/33089715/120100872-0b074d00-c17e-11eb-8b2c-a27592562ec8.png">


## 프래그먼트간 통신

프래그먼트간 직접 통신은 불가능하므로 다른 방법을 사용해야 한다.

### 1. AAC ViewModel 이용
ViewModel을 이용하면 Activity를 라이프사이클 오너로 등록하여 공통된 뷰모델을 이용할 수 있다.
+ Activity를 공유하는 여러 Fragment들은 Activity의 메모리를 공유할 수 있고 AAC의 ViewModel은 Activity의 Lifecycle보다 오래 살아있는 것이 보장되기 때문에 안전하게 공통의 Activity의 ViewModel을 사용하여 데이터 전달이 가능하다.

+ ViewModel
    ```kotlin
    class ViewModelPassActivity : AppCompatActivity() {
        val viewModel : PassingViewModel by viewModels()
    }

    class PassAFragment : Fragment() {
        val viewModel : PassingViewModel by activityViewModels()
    }

    class PassBFragment : Fragment() {
        val viewModel: PassingViewModel by activityViewModels()
    }
    ```

### 2. Interface를 이용한 리스너 구현
+ 인터페이스 하나와 onTextChange라는 메소드 선언
    ```java
    public interface FragmentListener {
        void onTextChange(String s);
    }
    ```
+ FragmentB에서 텍스트가 변경되면 이 인터페이스를 통해 데이터를 보낸다.
    ```java
    public class FragmentB2 extends Fragment {

        private EditText editText;
        private FragmentListener fragmentListener;

        @Nullable
        @Override
        public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
            View view = inflater.inflate(R.layout.fragment_b, container, false);
            editText = view.findViewById(R.id.input);
            return view;
        }

        @Override
        public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
            super.onViewCreated(view, savedInstanceState);
            editText.addTextChangedListener(new TextWatcher() {
                @Override
                public void beforeTextChanged(CharSequence s, int start, int count, int after) {

                }

                @Override
                public void onTextChanged(CharSequence s, int start, int before, int count) {

                }

                @Override
                public void afterTextChanged(Editable s) {
                    if(fragmentListener!=null){
                        fragmentListener.onTextChange(s.toString());
                    }
                }
            });
        }

        public void setFragmentListener(FragmentListener listener){
            this.fragmentListener = listener;
        }

        @Override
        public void onActivityCreated(@Nullable Bundle savedInstanceState) {
            super.onActivityCreated(savedInstanceState);
            if(getActivity() instanceof FragmentListener){
                this.fragmentListener = (FragmentListener) getActivity();
            }
        }
    }
    ```
+ Activity를 인터페이스 구현체로 하여 값이 변경 되었을 때 리스너를 통해 데이터의 변경을 통지한다.
    ```java
    public class InterfaceExamActivity extends AppCompatActivity implements FragmentListener{

        FragmentA2 fragmentA2;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_interface_exam);
            fragmentA2 = (FragmentA2) getSupportFragmentManager().findFragmentById(R.id.fragmentA);
        }


        @Override
        public void onTextChange(String s) {
            if(fragmentA2!=null){
                fragmentA2.setText(s);
            }
        }
    }
    ```

### 3. FragmentManager에 Bundle로 Data를 담아 전달
+ 전송하려는 Fragment
    ```kotlin
    //PassBundleFragment는 본인이 전달하고자 하는 Fragment class
    val bundle = Bundle()
    bundle.putString("key", "value")
    
    PassBundleFragment().arguments = bundle
    parentFragmentManager.beginTransaction()
        .replace(R.id.fragment_container_bundle, PassBundleFragment())
        .commit()
    ```

+ 전달 받는 Fragment
    ```kotlin
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
        ): View {
        var result = arguments?.getString("key")
        
            return inflater.inflate(R.layout.fragment_pass_b, container, false)
        }
    ```
## FragmentFactory
Fragment를 생성하는 쪽에서 특정한 데이터를 넘겨줘야 하는 경우에 다음과 같이 하면 앱이 강제종료되는 현상이 일어난다.
```kotlin
val index = 1
val newFragment = MyFragment(index)
```
Fragment를 상속 받을 땐 인자가 없는 기본 생성자를 반드시 포함해야 한다. Fragment는 자신만의 lifecycle을 가지고 있지만, 이는 Fragment를 호스팅하는 Activity의 lifecycle에 종속족이다. 따라서 메모리가 부족하거나 화면이 회전되는 등의 이벤트로 Activity가 재생성 될 때 Fragment도 함께 재생성이 된다. 이 때 호출되는 함수 instantiate()는 인자가 없는 기본 생성자로 Fragment를 생성한다.

### AndroidX 이전 - Bundle을 사용한 newInstance()
보통 아래와 같이 newInstance()를 사용해 객체를 생성한다.
```kotlin
class DetailsFragment : Fragment() {

    // ...
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        val index = arguments?.getInt(KEY_INDEX, 0) ?: 0
    }

    companion object {
        private const val KEY_INDEX = "index"
        
        /**
         * Create a new instance of DetailsFragment, initialized to
         * show the text at 'index'.
         */
        fun newInstance(index: Int): DetailsFragment {
            val f = DetailsFragment()

            // Supply index input as an argument.
            val args = Bundle()
            args.putInt(KEY_INDEX, index)
            f.arguments = args

            return f
        }
    }
}
```
1. Fragment 내부에 newInstance() 같은 이름의 static 메서드를 하나 만들고, 빈 생성자로 Fragment 인스턴스를 생성한다. 
2. Key-value 형식으로 Bundle에 데이터를 저장한다.
3. setArguments로 Fragment에 전달할 데이터를 설정한다.
4. Fragment에선 다시 getArguments() 메서드로 전달된 데이터를 꺼내서 사용할 수 있게 된다.

```kotlin
val index = 1
val newFragment = DetailsFragment.newInstance(index)
```
**이유??**

+ 재생성 때문, 재생성시 호출되는 함수 instantiate()는 인자가 없는 기본 생성자로 생성한다.
+ 프래그먼트가 재생성될 때 호출되는 메소드는 `Fragment[#instantiate(Context context, String fname, Bundle args)`이다. 매개 변수 중 Bundle이 존재한다. newInstance()를 통해 세팅했던 Bundle이 재생성될 때 다시 세팅되어 넘어온다. 따라서 Bundle을 통해 설정한 데이터들은 액티비티가 파기될 때 따러 onSaveInstanceState()하지 않아도 복구가 된다.
+ 위의 방법은 현재 deprecated 됐다.
### AndroidX 이후 - Factory method 패턴
1. 생성자로 인자를 받는 Fragment
```kotlin
class MyFragment(private val index: Int) : Fragment() {
    //...
}
```

2. FragmentFactory를 상속받은 후 클래스 이름에 따라 적절한 Fragment를 반환하도록 커스텀
```kotlin
class MyFragmentFactory(private val index: Int): FragmentFactory() {

    override fun instantiate(classLoader: ClassLoader, className: String): Fragment {
        return when (className) {
            MyFragment::class.java.name -> MyFragment(index)
            else -> super.instantiate(classLoader, className)
        }
    }
}
```

3. Activity에서 다음과 같이 fragment를 생성
```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        supportFragmentManager.fragmentFactory = MyFragmentFactory()
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, layoutId)
        
        val fragment = supportFragmentManager.fragmentFactory.instantiate(classLoader, MyFragment::class.java.name)
        supportFragmentManager.commit {
            add(containerId, fragment, MyFragment.TAG)
            addToBackStack(null)
        }
    }
```
+ 주의!! fragmentManager에 fragmentFactory 객체를 생성하는 부분은 반드시 lifecycle callback 중 super.onCreate() 보다 먼저 호출되어야 한다.
+ Fragment에 전달할 데이터가 없어서 빈 생성자를 사용한다면 FragmentFactory 를 커스텀 할 필요 없다.
+ koin 같은 라이브러리를 함께 사용하면 이 fragmentFactory를 직접 구현하지 않고도 더 간편하게 의존성 주입 처리를 할 수 있다.

--- 
### 출처
https://velog.io/@sysout-achieve/Android-Fragment%EA%B0%84-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%A0%84%EB%8B%AC-%EB%B0%A9%EB%B2%95%EB%93%A4
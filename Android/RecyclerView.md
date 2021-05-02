
# RecyclerView
보통 리스트뷰를 사용하는데 리스트뷰는 getview, view binding을 계속해서 하게 된다. 이는 성능 저하를 일으키고 따라서 viewholder 패턴을 권장한다. viewholder 패턴을 강제로 하는기 recyclerview이다. viewholder를 미리 몇 개 만들어 놓고 이를 재활용 한다.

## 주요 클래스
1. Adapter - 기존의 ListView에서 사용하는 Adapter와 같은 개념으로 데이터와 아이템에 대한 View 생성
2. ViewHolder - 재활용 View에 대한 모든 서브 뷰를 보유
3. LayoutManager - 아이템의 항목을 배치
4. ItemDecoration - 아이템 항목에서 서브뷰에 대한 처리
5. ItemAnimation - 아이템 항목이 추가, 제거되거나 정렬될 때 애니메이션 처리

### Adapter
리스트 뷰 : 데이터가 어디서 왔냐에 따라 BaseAdapter를 상속한 ArrayAdapter(배열로부터 데이터 가져올 때 사용), CursorAdapter(DB로 부터 가져올 때), SimpleAdapter(XML등으로 부터 가져올 때) 구분하여 사용

**리사이클러 뷰** : Universal한 Adapter를 사용하여 데이터 소스를 처리

- onCreateViewHolder(ViewGroup parent, int viewType) : 뷰 홀더를 생성하고 뷰를 붙여주는 부분
- onBindViewHolder(ListItemViewHolder holder, int position) : 재활용 되는 뷰가 호출하여 실행되는 메소드, 뷰 홀더를 전달하고 어댑터는 position의 데이터를 결합시킨다.
- getItemCount() : 데이터의 개수 반환

> 리스트 뷰가 사용했던 getView() 메소드는 매번 호출되면서 null 처리를 해줘야했다면, onCreateViewHolder는 새롭게 생성될 때만 불린다.
> Adapter : 100개의 채팅방 제목 이름이 담긴 리스트를 리사이클러 뷰에 바인딩 시켜주기 위한 사전작업이 이루어지는 객체

### ViewHolder
리스트 뷰 : 뷰 홀더 패턴을 권장. UI를 수정할 때 마다 부르는 findViewById()를 뷰홀더 패턴을 이용해 한 번만 함으로서 리스트 뷰의 지연을 초래하는 무거운 연산을 줄여줬다. 

**리사이클러 뷰** : 뷰 홀더 패턴을 항상 사용하도록 함으로서 해결(리스트 뷰나 리사이클러 뷰의 성능 차이는 메시하다, 차이점은 뷰홀더 패턴이 강제되는 것)

- 리사이클러뷰의 경우에는 예를들어 10개의 뷰 객체만 사용한다고 하면 10개의 뷰 객체들은 언제든 text 라던지 이미지가 바뀔 수 있다. 따라서 맨 처음 10개의 뷰 객체를 기억하고 있을 객체가 필요한데 이것이 ViewHolder이다.

    ```kotlin
    class MyViewHolder(view: View): RecyclerView.ViewHolder(view) {
        var textField = view.chat_title
    }
    ```
### LayoutManager
리스트 뷰 : 수직 스크롤만 가능, 수평으로 사용 불가

**리사이클러 뷰** : 수평 스크롤 지원, 다양한 타입의 리스트 지원, 커스텀 할 수 있도록 해줌, RecyclerView.LayoutManager 클래스 사용하면 된다.

- RecyclerView 내에 item view들의 크기를 측정하고 위치를 지정하는 역할, 언제 item view들을 재사용해야하는지에 대한 정책을 결정

### Item Decoration
클릭 이벤트 처리, RecyclerView 새로고침

## 호출 순서

1. 가장 먼저 실행되는 함수 getItemCount → 우리가 뿌려줄 데이터의 전체 길이를 리턴
2. 다음으로 onCreateViewHolder → ViewHolder가 생성되는 함수. ViewHolder 객체를 만들어 준다.
    - 맨 처음 화면에 보이는 전체 리스트 목록이 10개라면, 위아래 버퍼를 생각해서 13-15개 정도의 뷰 객체가 생성된다. 정확하게 말하자면 뷰 객체를 담고있는 ViewHolder가 생성된다. 그래서 onCreateViewHolder 함수는 13-15번 정도만 호출되고 더이상 호출되지 않는다

    - return 되는 곳에서 MyViewHolder의 생성자에 view 객체를 넘겨준다.

    - 이제는 재사용되는 뷰홀더(레이아웃)들에 데이터를 바인딩해주는 작업만 하면 된다.
3. onBindViewHolder가 호출된다. 생성된 뷰홀더에 데이터를 바인딩해주는 함수이다.
    - 예를들어 데이터가 스크롤 되어서 맨 위에 있던 뷰홀더 객체가 맨 아래로 이동한다면 그 레이아웃은 재사용하되 데이터는 새롭게 바뀐다.
    - onCreateViewHolder는 ViewHolder를 만들기 위해 13~15번 정도 밖에 호출되지 않지만 onBindViewHolder는 스크롤을 해서 데이터 바인딩이 새롭게 필요할 때마다 호출된다.
## ListView
### 일반적인 listview 사용법
```java
@Override
public View getView(final int position, View convertView, ViewGroup parent) {
    Holder holder = new Holder();
    View rowView = inflater.inflate(R.layout.item_list, null);
    holder.tv = (TextView) rowView.findViewById(R.id.text);
    holder.img = (ImageView) rowView.findViewById(R.id.image);
    holder.tv.setText(result[position]);
    holder.img.setImageResource(imageId[position]);
    rowView.setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View v) {
            // TODO Auto-generated method stub
            Toast.makeText(context, "You Clicked " + result[position], Toast.LENGTH_LONG).show();
        }
    });
    return rowView;
}
```
+ getView, 즉 ListView의 재사용성이 떨어진다.
    + 예를들어 아이템이 20개 있고 이를 스크롤 한다고 하면 스크롤시에도 getView() 함수는 계속해서 호출
+ 또 null 처리가 따로 없어 스크롤 할 때마다 inflate를 통해서 view의 create가 발생하겨 findViewById()도 함께 호출(메모리와 성능에 악영향)
+ 따라서 viewholder 개념 등장

### ListView + ViewHolder 패턴
``` java
@Override
public View getView(final int position, View convertView, ViewGroup parent) {
    // 최초에 convertView가 null이므로, inflate를 처리한다
    if (convertView == null) {
        // 전역으로 생성한 rootView에 inflate
        rootView = inflater.inflate(R.layout.item_list, null);

        // ViewHolder을 생성
        Holder holder = new Holder();
        holder.tv = (TextView) rowView.findViewById(R.id.text);
        holder.img = (ImageView) rowView.findViewById(R.id.image);

        // setTag : holder 임시 저장
        rootView.setTag(holder);
    } else {
        // rootView에 convertView를 셋팅
        rootView = convertView;
        // rootView에서 holder을 꺼내온다
        holder = (Holder) rootView.getTag();
    }

    holder.tv.setText(result[position]);
    holder.img.setImageResource(imageId[position]);
    rowView.setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View v) {
            // TODO Auto-generated method stub
            Toast.makeText(context, "You Clicked " + result[position], Toast.LENGTH_LONG).show();
        }
    });
    return rootView;
}
```
- convertView == null일 경우에만 inflate와 findViewById가 호출되어 view가 생성된다. 그리고 rootView의 setTag를 호출하여 생성된 ViewHolder를 임시 저장해둔다.
- 메모리에 문제가 없다면 최초 1회만 생성되고 이후 else 문을 통해서 getTag()를 호출하여 ViewHolder를 꺼내와서 ViewHolder에 접근이 가능한 형태가 만들어지게 된다.
- 강제적이지 않아 구현이 번거롭다.
- 커스텀이 많고 하나의 리스트에 다양한 viewholder를 만드는 것이 쉽지 않다.

    사진이 포함된 ViewHolder

    텍스트만 있는 ViewHolder

    오른쪽이 스크롤 되는 ListView가 포함된 ViewHolder

    - **따라서 RecyclerView 사용**

### 장점
- ListView는 간단하게 리스트를 만드는 부분에 있어 장점을 가지고 있다.(ex)텍스트만 있는 리스트)
- 간단한 아이템 형태를 만드는 경우에는 빠르게 적용이 가능한 ArrayAdapter를 제공한다.

### 단점
- 아이템의 애니메이션 처리가 쉽지 않다.
- 리스트 한 개 이상의 View가 필요한 경우가 있지만 커스텀으로 작업하기 쉽지 않다.
- ViewHolder 패턴을 강제적으로 사용하지 않으므로 고비용의 findViewById가 매번 호출될 수 있다.

## RecyclerView와 ListView의 차이점
||RecyclerView|ListView|
|:---:|---|---|
|ViewHolder|ViewHolder 패턴을 이용한다.|ViewHolder 패턴을 이용할 필요가 없다.|
|Item Layout|가로/세로/지그재그 방향 모두 지원|세로 방향만 지원|
|Item Animation|아이템 애니메이션을 처리하는 클래스가 있다.|아이템이 추가/제거될 때에 적용할 수 있는 애니메이션이 없다.|
|Adapter|데이터를 제공하기 위한 사용자 정의 구현이 필요|배열/DB 결과에 대한 ArrayAdapter/CursorAdapter와 같은 다양한 소스에 대한 어댑터가 존재한다.|
|Decoration|RecyclerView.ItemDecoration 객체를 사용하여 더 많은 구분선을 설정해야 한다.|목록의 개별 항목에 대한 클릭 이벤트에 바인딩하기 위핸 AdapterView.OnItemClickListener 인터페이스가 있다.|
|Click Detection|개별 터치 이벤트를 관리하지만 클릭 처리 기능이 내장되어 있지 않다.|목록의 개별 항목에 대한 클릭 이벤트에 바인딩하기 위한 AdapterView.OnItemClickListener 인터페이스가 있다.|

+ ListView : 각 아이템을 생성할 때 마다 뷰 바인딩을 계속해서 해주어 성능저하가 일어난다.
+ RecyclerView : ViewHolder 패턴을 강제로 구현하게 해서 뷰 바인딩을 한 번만 해주고, 이 후 아이템을 생성할 때 바인딩 된 뷰 객체를 재활용한다.Item에 대한 뷰의 변형, 애니메이션할 수 있다.
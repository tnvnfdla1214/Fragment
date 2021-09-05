# Fragment

다양한 크기의 화면을 가지는 단말기가 늘어남에 따라 한 화면에 여러 개의 화면 요소를 원하는 수요가 늘어가고 있습니다.
대표적으로 화면의 크기가 큰 태블릿 PC와 같이 화면의 크기가 큼에 따라 복잡한 레이아웃 구성과 뷰 위젯 배치들로 인해 기존의 Activity를 통한 레이아웃 구성만으로는 구현하기 버거운 면이 있었습니다. 

이를 커버하기 위해 나온것이 안드로이드 3.0(API 11)부터 추가된 개념인 프래그먼트(Fragment)입니다.
프래그먼트는 액티비티 내에서 화면 UI의 일부를 나타냅니다. 여러 개의 프래그먼트를 조합하여 액티비티가 출력하는 한 화면의 UI를 표현할 수 있으며 하나의 프래그먼트를 다른 액티비티에 재사용할 수 있습니다.
액티비티(Activity)처럼 하나의 독립된 모듈처럼 실행되기 때문에 액티비티와 연관된 생명주기를 가지고 있으며 액티비티 실행 중에도 화면에 동적으로 추가되거나 다른 Fragment로 교체가 가능합니다.

기본적으로 한 개의 액티비티에 들어가는 화면 요소를 Fragment 단위로 나누어 관리하기 때문에 레이아웃을 분리 관리할 수 있고, 액티비티의 화면 구성을 위한 레이아웃의 복잡도도 줄일 수 있습니다. 

***

<img src = "https://user-images.githubusercontent.com/48902047/132116741-94b95d23-0bbf-4c23-94a5-8807d2882aec.png" width="20%" height="20%"> <img src = "https://user-images.githubusercontent.com/48902047/132116759-a7f77087-7da6-4a3d-8aa6-ebc282a4987f.png" width="20%" height="20%">

+ [조치원 수호대](https://github.com/tnvnfdla1214/homemade_guardian) 
  + 메인화면 (바텀 네비게이션)
  + 채팅방 구성 화면 (게시물 썸네일 부분, 채팅 리스트)

***

### :lollipop: 설명 (채팅방 구성 화면)

#### 채팅 액티비티 구현

먼저 프레그먼트를 담을 채팅 액티비티를 구현 하였습니다.

ChatActivity에서는 프레그먼트를 배치하며 프레그먼트에 들어갈 기본 정보를 받아옵니다.

해당 유저에따라 게시물 썸네일 프레그먼트가 바뀌므로 게스트일때, 호스트일때, 게시물이 없을때를 나누어 배치합니다.

프래그먼트의 트랜젝션을 수행하기 위해서는 FragmentTrasaction에서 가져온 API를 활용해야 합니다. FragmentTrasaction 객체를 얻기 위해서는 FragmentManager 참조 객체가 필요합니다. 참조 객체를 얻기 위해서는 getFragmentManager() 함수 호출을 통해 가져올 수 있습니다. 

최종적으로 FragmentTrasaction 참조 객체를 얻기 위해서는 FragmentManager의 beginTransaction() 함수를 통해 얻을 수 있습니다. 

```Java
/*ChatActivity.Java*/

public class ChatActivity extends AppCompatActivity implements ChatFragment.RoomUidSetListener{

    private ChatFragment chatFragment;  //실제 채팅 프레그먼트
    private Host_Chat_MarketInfoFragment hostChat_MarketInfoFragment; //내 마켓 채팅 일때의 마켓정보 프레그먼트
    private Guest_Chat_MarketInfoFragment guestChat_MarketInfoFragment; //상대 마켓 채팅 일때의 마켓정보 프레그먼트
    private Nonepost_chat_MarketInfoFragment nonepost_chat_marketInfoFragment; //포스트의 정보가 없는 마켓정보 프레그먼트

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_chat);
        
        .
        .
        .
        // 채팅 프레그먼트
        chatFragment = ChatFragment.getInstance(To_User_Uid, ChatRoomListModel_RoomUid, MarketModel_Market_Uid,currentUser_Uid);
        getSupportFragmentManager()
                .beginTransaction()
                .replace(R.id.mainFragment, chatFragment )
                .commit();

        Firebasehelper = new FirebaseHelper(this);
        mcontext = this;


        //포스트 정보 프레그먼트
        DocumentReference docRef_USERS_HostUid = FirebaseFirestore.getInstance().collection("MARKETS").document(MarketModel_Market_Uid);
        docRef_USERS_HostUid.get().addOnSuccessListener(new OnSuccessListener<DocumentSnapshot>() {
            @Override
            public void onSuccess(DocumentSnapshot documentSnapshot) {
                marketModel = documentSnapshot.toObject(MarketModel.class);
                //if문 : 주어진 MarketModel_Market_Uid로 찾아간  marketModel이 null이면 없는 게시물이라는 것을 알려주기 위한 Nonepost_chat_MarketInfoFragment를 띄운다.
                //else, else if문 : currentUser_Uid == PostModel_Host_Uid 일 경우 Host_Chat_PostInfoFragment를 띄우고 아니라면 guest_Chat_PostInfoFragment를 띄운다.
                if(marketModel == null){
                    nonepost_chat_marketInfoFragment = new Nonepost_chat_MarketInfoFragment();
                    getSupportFragmentManager().beginTransaction().add(R.id.postinfoFragment, nonepost_chat_marketInfoFragment).commit();
                } else if(currentUser_Uid.equals(marketModel.getMarketModel_Host_Uid())){

                    hostChat_MarketInfoFragment = new Host_Chat_MarketInfoFragment();
                    getSupportFragmentManager().beginTransaction().add(R.id.postinfoFragment, hostChat_MarketInfoFragment).commit();
                    Bundle Marketbundle = new Bundle();
                    Marketbundle.putString("MarketModel_Market_Uid", MarketModel_Market_Uid);
                    Marketbundle.putString("To_User_Uid",To_User_Uid);
                    Marketbundle.putString("currentUser_Uid",currentUser_Uid);
                    hostChat_MarketInfoFragment.setArguments(Marketbundle);

                }
                else{
                    guestChat_MarketInfoFragment = new Guest_Chat_MarketInfoFragment();
                    getSupportFragmentManager().beginTransaction().add(R.id.postinfoFragment, guestChat_MarketInfoFragment).commit();
                    Bundle Marketbundle = new Bundle();
                    Marketbundle.putString("MarketModel_Market_Uid", MarketModel_Market_Uid);
                    guestChat_MarketInfoFragment.setArguments(Marketbundle);
                }
            }
        });
        
        .
        .
        .
    }
}
```
```Kotlin
/*activity_chat*/
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical"
    android:background="@color/pager_bg"
    >
    .
    .
    .
    
    <FrameLayout
        android:id="@+id/postinfoFragment"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <androidx.drawerlayout.widget.DrawerLayout
        android:id="@+id/drawer_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:openDrawer="start">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <FrameLayout
                android:id="@+id/mainFragment"
                android:layout_width="match_parent"
                android:layout_height="match_parent">

            </FrameLayout>

        </LinearLayout>

        <FrameLayout
            android:id="@+id/drawerFragment"
            android:layout_width="260dp"
            android:layout_height="match_parent"
            android:layout_gravity="right"
            android:background="@android:color/white">
        </FrameLayout>

    </androidx.drawerlayout.widget.DrawerLayout>

</LinearLayout>
```
#### 채팅 프레그먼트 구현

```Java
/*ChatFragment.java*/

//채팅룸 안의 프레그먼트
//채팅에서 어떻게 배열되는지 설정되는 기능(Chat_RecyclerView_Adapter 함수)
public class ChatFragment extends Fragment {

    public ChatFragment() {
    }

    public static final ChatFragment getInstance(String To_User_Uid, String RoomUiD,String MarketModel_Market_Uid,String currentUser_Uid) {
        ChatFragment chatFragment = new ChatFragment();
        Bundle bundle = new Bundle();
        bundle.putString("To_User_Uid", To_User_Uid);
        bundle.putString("RoomUid", RoomUiD);
        bundle.putString("MarketModel_Market_Uid", MarketModel_Market_Uid);
        bundle.putString("currentUser_Uid", currentUser_Uid);
        chatFragment.setArguments(bundle);
        return chatFragment;
    }
    
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_chat, container, false);
        .
        .
        .
        return view;
    }
}
```

#### 채팅 프레그먼트 구현

채팅의 실제 구현을 하는 프레그먼트

```Java
/*ChatFragment.java*/

public class ChatFragment extends Fragment {

    public ChatFragment() {
    }
    
     @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        chatActivity = (ChatActivity) getActivity();
    }
    
    public static final ChatFragment getInstance(String To_User_Uid, String RoomUiD,String MarketModel_Market_Uid,String currentUser_Uid) {
        ChatFragment chatFragment = new ChatFragment();
        Bundle bundle = new Bundle();
        bundle.putString("To_User_Uid", To_User_Uid);
        bundle.putString("RoomUid", RoomUiD);
        bundle.putString("MarketModel_Market_Uid", MarketModel_Market_Uid);
        bundle.putString("currentUser_Uid", currentUser_Uid);
        chatFragment.setArguments(bundle);
        return chatFragment;
    }
    
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_chat, container, false);
        .
        .
        .
        return view;
    }
}
```
#### 게스트 프레그먼트 구현 (호스트 프레그먼트, 정보없음 프레그먼트 모두 틀은 같습니다)

게시물 썸네일 정보를 연결해주는 프레그먼트

```Java
/*ChatFragment.java*/

public class Guest_Chat_MarketInfoFragment extends Fragment {

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        chatActivity = (ChatActivity) getActivity();
    }

    @Override
    public void onDetach() {
        super.onDetach();
        //이제 더이상 엑티비티 참초가안됨
        chatActivity = null;
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        .
        .
        .
        return View;
    }

}
```

***
### :pensive: 정리하면서 소감

하.....진짜 당시에는 하면서 진짜 자신이 대견스럽다고 생각했는데 진짜 너무 엉망진창 코딩이란걸 깨닿게 된다. 정리하는데도 너무 보기도 어렵다 

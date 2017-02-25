# 안드로이드 15장 

   여러가지 작업을 동시에 수행하기

* * *

## 03 애니메이션 만들어 보기

앞에서 스레드로 이미지를 바꿔주면서 에니메이션이 가능하다는 사실을 알게되었음

이미지가 변하는게 아니라 옆으로 이동하거나 회전하는 애니메이션을 만들때는 어떻게 해야할까?

강아지가 움직이는 영역 전체를 하나의 이미지로 잡아서 이미지를 변경시키면 된다

이때 각각의 이미지를 프레임이라고 하고 프로그램이 알아서 움직이도록 한다

![animation](https://github.com/misarm/android-project15/blob/master/ani.png?raw=true)

이런 방식이 트윈 애니메이션임

> 즉, 키가 되는 몇장의 프레임만 만들어도 나머지는 프로그램에서 계산하여 보이도록 하는 애니메이션임

표준 자바에서는 제공하지 않음, 안드로이드는 지원


<br>
<br>
>예제 코드
>acivity_main.xml 에서 화면 구성하기

```xml
    <Button
        android:text="물어와"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true"
        android:id="@+id/button" />

    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/button"
        android:layout_alignParentEnd="true"
        android:weightSum="1">

       <ImageView
            android:layout_width="63dp"
            android:layout_height="70dp"
            android:id="@+id/imageView"
            android:src="@drawable/dog"
            android:layout_alignTop="@+id/imageView"
            android:layout_alignParentEnd="true"
            android:layout_marginEnd="116dp"
            android:layout_weight="0.08" />
    </LinearLayout>

    <EditText
        android:layout_width="match_parent"
        android:inputType="textPersonName"
        android:ems="10"
        android:id="@+id/editText"
        android:gravity="top|center_vertical"
        android:layout_height="350dp"
        android:maxLines="10"
        android:layout_alignParentBottom="true"
        android:layout_alignParentStart="true"
        android:singleLine="false" />
```

![frame1](https://github.com/misarm/android-project15/blob/master/frame1.png?raw=true)



<br>
<br>
>애니메이션 액션정보 필요
>이미지가 어디로 어떻게 이동할 것인지 xml 코드로 미리 만들어둔다
>
> /res 폴더 안에 anim 폴더를 만들고 등록
>
> new -> Animation resource file -> translate.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate android:fromXDelta="0%"
        android:toXDelta="+300%"
        android:duration="1500"
        />
</set>
```

```java
public class MainActivity extends AppCompatActivity {

    ImageView imageView;

    EditText editText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        imageView = (ImageView) findViewById(R.id.imageView);

        editText = (EditText) findViewById(R.id.editText);

        Button button = (Button) findViewById(R.id.button);

        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {

                // AnimationUtils.loadAnimation 이용
                Animation anim = AnimationUtils.loadAnimation( getApplicationContext(), R.anim.translate );

                imageView.startAnimation( anim );

                editText.append( "애니메이션 시작됨. \n" );
            }
        });


    }
}
```

<br>
<br>
<br>

## 04 뷰를 상속하고 이벤트 처리하기

이미지를 손가락으로 터치 했을 시 이미지의 크기가 바뀌는 애니메이션 만들기
스레드를 이용해 이미지 변경


> /MyThread4/app/src/main/res/layout/activity_main.xml
> ![frame1](https://github.com/misarm/android-project15/blob/master/frame2.png?raw=true)


<br>
<br>

> /MyThread4/app/src/main/java/org/androidtown/org/mythread4/DogImageView.java

```java

 // View 클래스를 상속하면 터치 이벤트를 처리할 수 있고
 // 그 안에서도 스레드를 만들어 실행할수 있다
    
public class DogImageView extends ImageView{

    Handler handler = new Handler();

    //새로 추가한 두개의 생성자는 각각 다르게 사용할 예정

    //소스코드에서 new 이용해 직접 이미지뷰 객체를 만들때 사용
    public DogImageView(Context context) {
        super(context);
    }

    //화면에 들어갈 이미지뷰 위젯을 Xml 레이아웃에 추가 할때 사용
    public DogImageView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //return super.onTouchEvent(event);
        DogThread thread1 = new DogThread();
        thread1.start();

        return true;
    }

    //스레드 안에서 이미지를 변경하도록 한다
    class DogThread extends Thread {
        int stateIndex;

        ArrayList<Integer> images = new ArrayList<Integer>();

        public DogThread(){
            images.add(R.drawable.dog_standing);
            images.add(R.drawable.dog_running);
            images.add(R.drawable.dog_biting);
        }

        public void run(){
            stateIndex = 0;
            for(int i = 0; i < 9; i++){
                handler.post(new Runnable(){
                    @Override
                    public void run() {
                        setImageResource(images.get(stateIndex));
                    }
                });

                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e){
                    e.printStackTrace();
                }
                stateIndex++;
                if (stateIndex >= images.size()){
                    stateIndex = 0;
                }
            }
        }
    }
}
```

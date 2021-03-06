






20200825 (화) 
### 수업 목차
#### 1. 메인 컴포넌트 4 | Content Provider (Data 공유)

#### 2. ProviderTest 프로젝트 | Data 공유 예제

예제1) [외부 어플리케이션 데이터 공유받기](#예제1-Content-Provider로-외부-어플리케이션-데이터-공유받기)<br/>

예제2)[ 내부 어플리케이션 데이터 공유하기](#예제2-Content-Provider로-내부-어플리케이션-데이터-공유하기) <br/> 







<br/>
<br/>


#  1. Content Provider | Data 공유
<br/>

> Content Provider? <br/>

#### content provider == 대리자 역할
 안드로이드는 기본적으로 데이터를 다른 어플리케이션과 공유하지 않음. <br/> 즉, 다이렉트로 해당 데이터에 접근 불가능.<br/>
 따라서 데이터를 소유한 쪽에서 content provider를 이용해서 공유하도록 한다. 
<br/><br/>

> 데이터 공유 예시

카톡에서 내 폰의 전화번호부나 사진첩에 있는 정보를 가져다가 사용
<br/><br/>

> 데이터 공유 종류

1. 내 쪽에서 다른 쪽으로 제공
2. 다른 쪽이 오픈한 것을 내 쪽에서 사용
<br/><br/>

> 공유 방법<br/>

1.  데이터를 공유하는 측에서, content provider를 외부에서 오픈할 수 있는 URI(자원의 위치 정보를 웹 주소처럼 표현)를 오픈해놓음.
2.  content provider는 permission 지정 가능. 따라서 사용하려는 권한을 manifest.xml에 등록해야 사용 가능. 하나의 권한으로는 하나의 작업만 가능
	- 쓰기 권한
	- 읽기 권한
3. 요청하는 쪽에서는 content resolver로 provider에게 데이터를 요청. 'db에 insert, update, delete를 해줘' 

4.  요청을 받는 쪽에서는,  요청을 받은 content provider가 Dao의 메서드를 호출해서 db작업을 수행한 뒤, 요청한 어플리케이션 쪽으로 작업 결과를 전달. 
<br/><br/>

> 요청 
- content provider basics<br/>
<img src="https://user-images.githubusercontent.com/62331803/91110717-9e749480-e6b9-11ea-93c5-a0290fcbce61.png" width="70%">

<br/>

- querying a content provider<br/>
<img src="https://user-images.githubusercontent.com/62331803/91110907-1a6edc80-e6ba-11ea-9c31-fd56b22f51e3.png" width="70%">

 <br/>

- querying 샘플코드<br/>
<img src="https://user-images.githubusercontent.com/62331803/91110938-2c507f80-e6ba-11ea-8fe3-8a6d528cbfc9.png" width="70%">


<br/>

> URI 표현 (manifest에 open) <br/>

#### 예시) content://package_name.provider_name/table내의 items or item/13

<img src="https://user-images.githubusercontent.com/62331803/91110824-e5fb2080-e6b9-11ea-87e4-6e283645e7f2.png" width="70%">


<br/>

> data model <br/>

SQLite를 기반으로 작동된다. 즉, cursor 객체를 반환함.
따라서 Room database를 사용할 경우 SQLite database로 전환하거나, 반환 타입을 cursor로 변환하여 사용함.<br/>

<img src="https://user-images.githubusercontent.com/62331803/91110785-d24fba00-e6b9-11ea-8121-2d3454257c59.png" width="70%">

<br/>

> 데이터를 얻기 위해 필요한 정보/제공자가 open해야하는 정보 <br/>

- provider URI
- 컬럼 이름
- 컬럼에 대한 데이터 타입

<br/>

> 요청 예시<br/>

```java
ContentResover cr = getContentResolver();
Cursor c = cr.query(uri,new String[]("name","tel",..);//uri, 컬럼명, where 절, 물음표 처리...
if(c.moveToFirst()){
	do{
		c.getInt(0),...
	}while(c.moveToNext);//데이터가 존재할 동안 다음 줄로 이동해서 데이터 추출
}

```

<br/>

# 2. ProviderTest 프로젝트 | Data 공유 예제

> Content Provider 만들기<br/>

- 구현할 메서드<br/>
<img src="https://user-images.githubusercontent.com/62331803/91114164-38d8d600-e6c2-11ea-9484-d96c9f2df858.png" width="70%">


- Uri 등록 <br/>
<img src="https://user-images.githubusercontent.com/62331803/91114170-3d04f380-e6c2-11ea-9753-d4f39bdb84ea.png" width="70%">

<br/>

## 예제1) Content Provider로 외부 어플리케이션 데이터 공유받기

###  :: 안드로이드 phonebook 데이터 활용하기
<br/>

> Create new contacts <br/>

<img src="https://user-images.githubusercontent.com/62331803/91110201-0aee9400-e6b8-11ea-86ff-addc921f1209.png" width="40%" height="30%">

<br/>

> Manifest.xml <br/>

읽기/쓰기 permission 추가
```xml
<uses-permission android:name="android.permission.READ_CONTACTS"></uses-permission>  
<uses-permission android:name="android.permission.WRITE_CONTACTS"></uses-permission>
```

<br/>

> MainActivity.java
-  permission 존재유무 확인

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    //For device above MarshMellow
    boolean permission = getSMSPermission();
    if (permission) {
        Toast.makeText(this, "obtain contact permission", Toast.LENGTH_SHORT).show();
    }
} else {
    Toast.makeText(this, "failed to obtain contact permission", Toast.LENGTH_SHORT).show();
}
```

-  phonebook에 쓰기 작업

```java
// 1. content value 객체를 생성
ContentValues values = new ContentValues();
String contactId = "aaa";

//2. contentValues 객체에 db 작업할 id를 설정
values.put(ContactsContract.RawContacts.CONTACT_ID, contactId); // data table의 _id 컬럼에 접근할 레퍼런스

//3. contentValues 객체를 이용해 DB에 id저장하고, Uri를 반환 받음
Uri contactUri = getContentResolver().insert(ContactsContract.RawContacts.CONTENT_URI, values);

//4.받은 uri에서 id값만 파싱. 안드로이드에서는 long타입으로 받음
//_id 컬럼의 값 읽어와 할당
long contactId_l = ContentUris.parseId(contactUri);

//5.데이터 처리
//5-1. ContentValues의 데이터를 모두 삭제(새 데이터 삽입을 위함)
values.clear();
//5-2. ContentValues에 저장할 데이터의 id 저장
values.put(ContactsContract.RawContacts.Data.RAW_CONTACT_ID, contactId_l);
//5-3. (선택) 데이터 마임 타입(프로바이더가 제공하는 데이터 타입) 저장
values.put(ContactsContract.Data.MIMETYPE, Phone.CONTENT_ITEM_TYPE);
//5-4. 전화번호 저장
values.put(Phone.NUMBER, "010-0000-0000");
//5-5. 전화타입 저장
values.put(Phone.TYPE, Phone.TYPE_MOBILE);
//5-6. 사용자 식별 레이블 저장
values.put(Phone.LABEL, "aaa");

//6. setting한 values를 db 테이블에 삽입
Uri dataUri = getContentResolver().insert(ContactsContract.Data.CONTENT_URI, values);
```

-  phonebook에서 읽기 작업

```java
Cursor c = getContentResolver().query(ContactsContract.Data.CONTENT_URI,
    new String[] {
        ContactsContract.Contacts.Data._ID, ContactsContract.CommonDataKinds.Phone.NUMBER, ContactsContract.CommonDataKinds.Phone.TYPE, ContactsContract.CommonDataKinds.Phone.LABEL
    },
    null, null, null);
String id = null, number = null, type = null, label = null;
int num;

//Cursor의 시작 위치로 이동해 데이터(튜플)를 한 줄씩 읽음
if (c.moveToFirst()) {
    do {
        //현재 튜플의 number와 type column의 값을 읽음
        number = c.getString(c.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER)); //전화번호. COLUMN의 idx로 검색.
        num = c.getShort((c.getColumnIndex(ContactsContract.CommonDataKinds.Phone.TYPE))); //전화타입

        //전화타입(정수) 값을 String(문자열)으로 변환
        switch (num) {
            case 1:
                type = "HOME";
                break;
            case 2:
                type = "MOBILE";
                break;
            case 3:
                type = "WORK";
                break;
            default:
                type = "ETC";
        }
        label = c.getString(c.getColumnIndex(Phone.LABEL));
        String str = "label:" + label + ", number:" + number + ", type:" + type;
        Toast.makeText(this, str, Toast.LENGTH_SHORT).show();

    } while (c.moveToNext());
}
```

- permission 획득

```java
public boolean getSMSPermission() {
    boolean hasPermission = (
        (ContextCompat.checkSelfPermission(this,
            Manifest.permission.READ_CONTACTS) == PackageManager.PERMISSION_GRANTED) &&
        (ContextCompat.checkSelfPermission(this,
            Manifest.permission.WRITE_CONTACTS) == PackageManager.PERMISSION_GRANTED)

    );
    if (!hasPermission) {
        ActivityCompat.requestPermissions(this, new String[] {
            Manifest.permission.READ_CONTACTS, Manifest.permission.WRITE_CONTACTS
        }, 1);
    }
    return hasPermission;
}

@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    switch (requestCode) {
        case 1: {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                Toast.makeText(this, "obtain contact permission", Toast.LENGTH_SHORT).show();
            }
        }
    }
}
```
<br/>

<details>
<summary>MainActivity.java 전체코드 보기</summary>

```java
package com.example.providertest;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.content.ContentUris;
import android.content.ContentValues;
import android.content.pm.PackageManager;
import android.database.Cursor;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.provider.ContactsContract;
import android.provider.ContactsContract.CommonDataKinds.*;
import android.widget.Toast;


public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //<permission 존재 유무 확인>
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            //For device above MarshMellow
            boolean permission = getSMSPermission();
            if (permission) {
                Toast.makeText(this, "obtain contact permission", Toast.LENGTH_SHORT).show();
            }
        } else {
            Toast.makeText(this, "failed to obtain contact permission", Toast.LENGTH_SHORT).show();
        }

        // <phonebook에 쓰기 작업>
        // 1. content value 객체를 생성
        ContentValues values = new ContentValues();
        String contactId = "aaa";

        //2. contentValues 객체에 db 작업할 id를 설정
        values.put(ContactsContract.RawContacts.CONTACT_ID, contactId); // data table의 _id 컬럼에 접근할 레퍼런스

        //3. contentValues 객체를 이용해 DB에 id저장하고, Uri를 반환 받음
        Uri contactUri = getContentResolver().insert(ContactsContract.RawContacts.CONTENT_URI, values);

        //4.받은 uri에서 id값만 파싱. 안드로이드에서는 long타입으로 받음
        //_id 컬럼의 값 읽어와 할당
        long contactId_l = ContentUris.parseId(contactUri);

        //5.데이터 처리
        //5-1. ContentValues의 데이터를 모두 삭제(새 데이터 삽입을 위함)
        values.clear();
        //5-2. ContentValues에 저장할 데이터의 id 저장
        values.put(ContactsContract.RawContacts.Data.RAW_CONTACT_ID, contactId_l);
        //5-3. (선택) 데이터 마임 타입(프로바이더가 제공하는 데이터 타입) 저장
        values.put(ContactsContract.Data.MIMETYPE, Phone.CONTENT_ITEM_TYPE);
        //5-4. 전화번호 저장
        values.put(Phone.NUMBER, "010-0000-0000");
        //5-5. 전화타입 저장
        values.put(Phone.TYPE, Phone.TYPE_MOBILE);
        //5-6. 사용자 식별 레이블 저장
        values.put(Phone.LABEL, "aaa");

        //6. setting한 values를 db 테이블에 삽입
        Uri dataUri = getContentResolver().insert(ContactsContract.Data.CONTENT_URI, values);


        //<phonebook으로부터 읽기 작업>
        Cursor c = getContentResolver().query(ContactsContract.Data.CONTENT_URI,
            new String[] {
                ContactsContract.Contacts.Data._ID, ContactsContract.CommonDataKinds.Phone.NUMBER, ContactsContract.CommonDataKinds.Phone.TYPE, ContactsContract.CommonDataKinds.Phone.LABEL
            },
            null, null, null);
        String id = null, number = null, type = null, label = null;
        int num;

        //Cursor의 시작 위치로 이동해 데이터(튜플)를 한 줄씩 읽음
        if (c.moveToFirst()) {
            do {
                //현재 튜플의 number와 type column의 값을 읽음
                number = c.getString(c.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER)); //전화번호. COLUMN의 idx로 검색.
                num = c.getShort((c.getColumnIndex(ContactsContract.CommonDataKinds.Phone.TYPE))); //전화타입

                //전화타입(정수) 값을 String(문자열)으로 변환
                switch (num) {
                    case 1:
                        type = "HOME";
                        break;
                    case 2:
                        type = "MOBILE";
                        break;
                    case 3:
                        type = "WORK";
                        break;
                    default:
                        type = "ETC";
                }
                label = c.getString(c.getColumnIndex(Phone.LABEL));
                String str = "label:" + label + ", number:" + number + ", type:" + type;
                Toast.makeText(this, str, Toast.LENGTH_SHORT).show();

            } while (c.moveToNext());
        }
    }

    //permission 획득
    public boolean getSMSPermission() {
        boolean hasPermission = (
            (ContextCompat.checkSelfPermission(this,
                Manifest.permission.READ_CONTACTS) == PackageManager.PERMISSION_GRANTED) &&
            (ContextCompat.checkSelfPermission(this,
                Manifest.permission.WRITE_CONTACTS) == PackageManager.PERMISSION_GRANTED)

        );
        if (!hasPermission) {
            ActivityCompat.requestPermissions(this, new String[] {
                Manifest.permission.READ_CONTACTS, Manifest.permission.WRITE_CONTACTS
            }, 1);
        }
        return hasPermission;
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case 1: {
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    Toast.makeText(this, "obtain contact permission", Toast.LENGTH_SHORT).show();
                }
            }
        }
    }
}

```

</details>
<br/>

> 결과 화면 <br/>

저장한 데이터 toast로 출력<br/>

<img src="https://user-images.githubusercontent.com/62331803/91112628-95d28d00-e6be-11ea-8e05-6137fb77cff6.png" width="35%">


<br/>

## 예제2) Content Provider로 내부 어플리케이션 데이터 공유하기

> Member

<details>
<summary>코드보기</summary>

```java
package com.example.providertest;


import androidx.room.Entity;
import androidx.room.PrimaryKey;

//VO 클래스(테이블)
@Entity
public class Member {
    @PrimaryKey(autoGenerate = true)
    public int _id;
    public String name;
    public String tel;

    public Member() {}

    public Member(int _id, String name, String tel) {
        this._id = _id;
        this.name = name;
        this.tel = tel;
    }

    @Override
    public String toString() {
        return "_id=" + _id + "     name= " + name + "      tel=" + tel;
    }
}
```

</details>
<br/>

> MemberDao

<details>
<summary>코드보기</summary>

```java
package com.example.providertest;

import androidx.room.Dao;
import androidx.room.Delete;
import androidx.room.Insert;
import androidx.room.Query;
import androidx.room.Update;

import java.util.List;

//JPARepository 역할
@Dao
public interface MemberDao {
    @Insert
    void insert(Member m);

    @Query("select * from member")
    List < Member > selectAll();

    @Query("select * from member where _id=(:id)") //_id 처럼 언더바 쓰면 오류남
    Member selectById(int id);

    @Delete
    void delete(Member m);

    @Update
    void update(Member m);
}
```
</details>
<br/>

> AppDatabase

<details>
<summary>코드보기</summary>

```java
package com.example.providertest;

import android.content.Context;

import androidx.room.Database;
import androidx.room.Room;
import androidx.room.RoomDatabase;

@Database(entities = {
    Member.class
}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    private static AppDatabase appDatabase; //database
    public abstract MemberDao memberDao(); //Repository 객체

    public static AppDatabase getInstance(Context context) {
        //싱글톤으로 생성

        if (appDatabase == null) {
            appDatabase = Room.databaseBuilder(context, AppDatabase.class, "app_db").build();
        }
        return appDatabase;
    }

    public static void delDB() {
        appDatabase = null;
    }
}
```
</details>

<br/>

> build.gradle(Module:app) 수정

<details>
<summary>코드보기</summary>

```gradle
def room_version = "2.2.5"  
  
implementation "androidx.room:room-runtime:$room_version"  
annotationProcessor "androidx.room:room-compiler:$room_version" // For Kotlin use kapt instead of annotationProcessor  
  
// optional - Kotlin Extensions and Coroutines support for Room  implementation "androidx.room:room-ktx:$room_version"  
  
// optional - RxJava support for Room  
implementation "androidx.room:room-rxjava2:$room_version"  
  
// optional - Guava support for Room, including Optional and ListenableFuture  
implementation "androidx.room:room-guava:$room_version"  
  
// Test helpers  
testImplementation "androidx.room:room-testing:$room_version"  
  
//viewModel 의존성  
implementation "androidx.lifecycle:lifecycle-extensions:2.2.0"
```
</details>

<br/>

> MyContentProvider
- provider 생성 <br/>
<img src="https://user-images.githubusercontent.com/62331803/91113377-6a50a200-e6c0-11ea-93fd-98a883e09bd2.png" width="40%">

- provider uri 설정<br/>
<img src="https://user-images.githubusercontent.com/62331803/91113431-88b69d80-e6c0-11ea-9b60-b53a0c9585aa.png" width="35%">



- manifest 파일 확인<br/>
```xml
<provider
            android:name=".MyContentProvider"
            android:authorities="com.example.providertest.provider"
            android:enabled="true"
            android:exported="true">
</provider>
```
<br/>

> SQLiteSupport | DB 작업을 위한 클래스 <br/>


```java
package com.example.providertest;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;

import androidx.sqlite.db.SupportSQLiteDatabase;
import androidx.sqlite.db.SupportSQLiteOpenHelper;

import java.io.IOException;

public class SQLiteSupport {
    private AppDatabase database; //db파일 오픈
    private SupportSQLiteDatabase db; //room db를 SQLite db로 사용할 수 있게 변환

    public SQLiteSupport(Context context) {
        database = AppDatabase.getInstance(context);
    }

    public void open() {
        SupportSQLiteOpenHelper helper = database.getOpenHelper(); //헬퍼 객체 생성
        db = helper.getWritableDatabase();
    }

    public Cursor select(String t, String where) {
        String sql = "select * from " + t; //기본 sql문
        if (where != null && !where.equals("")) { //where절 설정된 경우
            sql += " where " + where;
        }
        return db.query(sql);
    }
    public long insert(String t, ContentValues cv) {
        return db.insert(t, 0, cv); //id 반환
    }
    public int update(String t, ContentValues cv, String where, String[] args) { //cv: 수정할 data, 컬럼..
        return db.update(t, 0, cv, where, args); //처리된 튜플 수 반환
    }
    public int delete(String t, String where, String[] args) {
        return db.delete(t, where, args); //처리된 튜플 수 반환
    }
    public void close() {
        try {
            db.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

<br/>

> MyContentProvider.java

<details>
<summary>코드 보기</summary>

```java
package com.example.providertest;

import android.content.ContentProvider;
import android.content.ContentValues;
import android.content.UriMatcher;
import android.database.Cursor;
import android.net.Uri;
import android.text.TextUtils;

public class MyContentProvider extends ContentProvider {
    private SQLiteSupport db;
    //cv의 Uri 상수로 설정
    public final static Uri CONTENT_URI = Uri.parse("content://com.example.providertest.provider/member");
    private final static int DATAS = 1;
    private final static int DATA_ID = 2;

    //테이블 이름
    private final static String TABLE_NAME = "member";

    //요청 uri 패턴 구분을 위한 urimatcher
    private final static UriMatcher urimatcher;
    static {
        urimatcher = new UriMatcher(UriMatcher.NO_MATCH);
        urimatcher.addURI("com.example.providertest.provider", "member", DATAS); //uri만
        urimatcher.addURI("com.example.providertest.provider", "member/#", DATA_ID); //uri+id ex)/member/3
    }

    public MyContentProvider() {}

    @Override
    public boolean onCreate() {
        db = new SQLiteSupport(getContext()); //db 작업을 위한 sqlitesupport 객체 생성
        db.open();
        return false;
    }

    // 요청한 uri 패턴에 따라 MIME 타입을 반환(집합 형태 or 단일 형태)
    // mime type 예시: mp3,mp4...
    // 현재 cv가 가진 데이터가 집합인지 단일행인지를 알려주는 역할
    @Override
    public String getType(Uri uri) {
        switch (urimatcher.match(uri)) {
            case DATAS: //집합 데이터 (dir)
                return "com.example.providertest.provider/dir";
            case DATA_ID: //단일행 데이터 (item)
                return "com.example.providertest.provider/item";
        }
        return null;
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) { //uri, where절, ? 인자 지정 배열
        int cnt = 0;
        switch (urimatcher.match(uri)) {
            case DATAS: //기본형: com.example.providertest.provider/member
                cnt = db.delete(TABLE_NAME, selection, selectionArgs);
                break;
            case DATA_ID: //조건형: com.example.providertest.provider/member/12
                //where절 없이 사용하는 대신 해당 번호를 where절에 설정해줌
                String _id = uri.getPathSegments().get(1); //0(테이블명):member, 1(조건):12
                cnt = db.delete(TABLE_NAME, "_id=" + _id + (!TextUtils.isEmpty(selection) ? " and " + selection : ""), selectionArgs);
                //!TextUtils.isEmpty(selection) ==> where절이 비지 않은 경우
                break;
        }
        getContext().getContentResolver().notifyChange(uri, null); //변화가 있다는 것을 알림.
        return 0;
    }


    @Override
    public Uri insert(Uri uri, ContentValues values) {
        long id = db.insert(TABLE_NAME, values);
        Uri u = null;
        if (id > 0) { //반환된 id가 1개 이상일 경우(정상 동작한 경우)
            u = Uri.withAppendedPath(CONTENT_URI, id + "");
            getContext().getContentResolver().notifyChange(u, null);
        }
        return u;
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection,
        String[] selectionArgs, String sortOrder) {
        Cursor c = null;
        switch (urimatcher.match(uri)) {
            case DATAS: //집합 데이터
                c = db.select(TABLE_NAME, selection);
                break;
            case DATA_ID: //단일행 데이터::조건을 붙여준다
                String _id = uri.getPathSegments().get(1); //0(테이블명), 1(조건)
                c = db.select(TABLE_NAME, "_id=" + _id + (!TextUtils.isEmpty(selection) ? " and " + selection : ""));
                //!TextUtils.isEmpty(selection) ==> where절이 비지 않은 경우
                break;
        }
        return c;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection,
        String[] selectionArgs) {
        int cnt = 0;
        switch (urimatcher.match(uri)) {
            case DATAS: //기본형: com.example.providertest.provider/member
                cnt = db.update(TABLE_NAME, values, selection, selectionArgs);
                break;
            case DATA_ID: //조건형: com.example.providertest.provider/member/12
                String _id = uri.getPathSegments().get(1); //0(테이블명):member, 1(조건):12
                cnt = db.update(TABLE_NAME, values, "_id=" + _id + (!TextUtils.isEmpty(selection) ? " and " + selection : ""), selectionArgs);
                break;
        }
        getContext().getContentResolver().notifyChange(uri, null); //변화가 있다는 것을 알림.
        return cnt;
    }
}
```

</details>

<br/>


> activity_provider_test.xml <br/>

<img src="https://user-images.githubusercontent.com/62331803/91120947-beb04d80-e6d1-11ea-99e2-7c470ccdd03c.png" width="60%"> 
<br/>


> ProviderTestActivity.java

<details>
<summary>코드 보기</summary>

```java
package com.example.providertest;

import androidx.appcompat.app.AppCompatActivity;

import android.content.ContentValues;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.EditText;
import android.widget.ListView;

import java.util.ArrayList;

public class ProviderTestActivity extends AppCompatActivity {
    private EditText name_et;
    private EditText tel_et;
    private ListView listView;
    private ArrayAdapter < Member > adapter;
    private ArrayList < Member > list;
    private Uri uri = MyContentProvider.CONTENT_URI;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_provider_test);
        name_et = findViewById(R.id.name);
        tel_et = findViewById(R.id.tel);
        listView = findViewById(R.id.lv1);
        list = new ArrayList < > ();

        adapter = new ArrayAdapter < > (this, android.R.layout.simple_list_item_1, list);
        listView.setAdapter(adapter);
        getAll();
    }

    public void getAll() {
        list.clear();
        Cursor c = getContentResolver().query(uri, null, null, null, null);
        if (c.moveToFirst()) {
            do {
                list.add(new Member(c.getInt(0), c.getString(1), c.getString(2)));
            } while (c.moveToNext());
        }
        adapter.notifyDataSetChanged();
    }

    public void onSave(View view) {
        String n = name_et.getText().toString();
        String t = tel_et.getText().toString();
        ContentValues cv = new ContentValues();
        cv.put("name", n);
        cv.put("tel", t);
        getContentResolver().insert(uri, cv);
        name_et.setText("");
        tel_et.setText("");
        getAll();
    }

    public void onEdit(View view) {

    }
}
```

</details>
<br/>

> 결과화면<br/>

<img src="https://user-images.githubusercontent.com/62331803/91124138-86ad0880-e6d9-11ea-9ac6-b68ac2ebbea4.png" width="40%">

<br/><br/>




## HomeWork 
다른 어플리케이션에서 MyContentProvider로 데이터 공유 요청





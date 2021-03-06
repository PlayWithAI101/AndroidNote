






20200826 (수) 
### 수업 목차
#### 1. NetTest 프로젝트 | 웹 연동

예제1) [UI에 웹사이트 띄우기](#예제1-UI에-웹사이트-띄우기)<br/>

예제2) [웹에서 xml 파일 불러와서 사용하기](#예제2-웹에서-xml-파일-불러와서-사용하기)<br/>

예제3) [웹에서 json 파일 불러와서 사용하기](#예제3-웹에서-json-파일-불러와서-사용하기)<br/>

예제4) [Socket 통신](#예제4-Socket-통신)<br/>


#### 2. WebApp 프로젝트 | 웹 어플리케이션 연동 (진행중)

예제1) [로그인 기능](#예제1-로그인)<br/>







<br/>
<br/>


#  1. NetTest 프로젝트 | 안드로이드 웹 연동
<br/>

## 예제1) UI에 웹사이트 띄우기

> manifest.xml  <br/>

인터넷 사용 permission 추가
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```
<br/>

> activity_main.xml<br/>

<img src="https://user-images.githubusercontent.com/62331803/91241093-d42f8100-e77e-11ea-86b6-38c6ad4d0e77.png" width="70%">

<br/>

>  MainActivity.java

<details>
<summary> 코드보기 </summary>

```java
package com.example.nettest;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.view.View;
import android.webkit.WebView;
import android.widget.EditText;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
    private WebView webView;
    private EditText addr;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        webView = findViewById(R.id.wv);
        addr = findViewById(R.id.addr);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            //For device above MarshMellow
            boolean permission = getInternetPermission();
            if (permission) {
                Toast.makeText(this, "obtain contact permission", Toast.LENGTH_SHORT).show();
            }
        } else {
            Toast.makeText(this, "failed to obtain contact permission", Toast.LENGTH_SHORT).show();
        }
    }

    public void onGo(View view) {
        String url = addr.getText().toString();
        webView.loadUrl(url);
        addr.setText("");
    }

    public boolean getInternetPermission() {
        boolean hasPermission = (
            (ContextCompat.checkSelfPermission(this,
                Manifest.permission.INTERNET) == PackageManager.PERMISSION_GRANTED));
        if (!hasPermission) {
            ActivityCompat.requestPermissions(this, new String[] {
                Manifest.permission.INTERNET
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
                    Toast.makeText(this, "obtain internet permission", Toast.LENGTH_SHORT).show();
                }
            }
        }
    }

}
```

</details>

<br/>

> 결과화면 <br/>

- 주소형식: https://주소~ <br/>
<img src="https://user-images.githubusercontent.com/62331803/91240967-769b3480-e77e-11ea-85a3-724a72957abf.png" width="50%">

<br/>

## 예제2) 웹에서 xml 파일 불러와서 사용하기

> get API key<br/>

https://openweathermap.org/price

<br/>

<img src="https://user-images.githubusercontent.com/62331803/91241233-41dbad00-e77f-11ea-82db-b768ad7591e5.png" width="80%">

<br/>

> 샘플 xml파일 <br/>

<img src="https://user-images.githubusercontent.com/62331803/91243842-d6490e00-e785-11ea-8f0c-ebd7fc5c941b.png" width="80%">
<br/>

> WeatherInfo.java | VO클래스

<details>
<summary>코드보기</summary>

```java
package com.example.nettest;

/**
 * Created by usr on 2015-08-28.
 */
public class WeatherInfo {
    private String city;
    private String temperature;
    private String humidity;
    private String clouds;

    public WeatherInfo() {}
    public WeatherInfo(String city, String temperature, String humidity, String clouds) {
        this.city = city;
        this.temperature = temperature;
        this.humidity = humidity;
        this.clouds = clouds;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getTemperature() {
        return temperature;
    }

    public void setTemperature(String temperature) {
        this.temperature = temperature;
    }

    public String getHumidity() {
        return humidity;
    }

    public void setHumidity(String humidity) {
        this.humidity = humidity;
    }

    public String getClouds() {
        return clouds;
    }

    public void setClouds(String clouds) {
        this.clouds = clouds;
    }

    @Override
    public String toString() {
        return "WeatherInfo{" +
            "city='" + city + '\'' +
            ", temperature='" + temperature + '\'' +
            ", humidity='" + humidity + '\'' +
            ", clouds='" + clouds + '\'' +
            '}';
    }
}
```

</details>

<br/>

> activity_main2.xml <br/>

<img src="https://user-images.githubusercontent.com/62331803/91242914-9b45db00-e783-11ea-8398-acd8e01a9bea.png" width="70%">

<br/>

> MainActivity2.java

<details>
<summary>코드보기</summary>

```java
package com.example.nettest;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.os.StrictMode;
import android.widget.TextView;
import android.widget.Toast;

import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.xml.sax.SAXException;

import java.io.IOException;
import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLConnection;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;

public class MainActivity2 extends AppCompatActivity {
    private TextView tv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);
        tv = findViewById(R.id.tv);

        // 네트워크를 메인쓰레드에서 작업하면 안됨.
        // 정석으로는 스레드를 파생하여 실행해야 한다.
        // 이렇게 작업하는 대신 편법으로 하게끔 하는 코드
        StrictMode.ThreadPolicy policy = new StrictMode.ThreadPolicy.Builder().permitAll().build();
        StrictMode.setThreadPolicy(policy);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            //For device above MarshMellow
            boolean permission = getInternetPermission();
            if (permission) {
                Toast.makeText(this, "obtain contact permission", Toast.LENGTH_SHORT).show();
            }
        } else {
            Toast.makeText(this, "failed to obtain contact permission", Toast.LENGTH_SHORT).show();
        }
        show();
    }

    public boolean getInternetPermission() {
        boolean hasPermission = (
            (ContextCompat.checkSelfPermission(this,
                Manifest.permission.INTERNET) == PackageManager.PERMISSION_GRANTED));
        if (!hasPermission) {
            ActivityCompat.requestPermissions(this, new String[] {
                Manifest.permission.INTERNET
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
                    Toast.makeText(this, "obtain internet permission", Toast.LENGTH_SHORT).show();
                }
            }
        }
    }

    public void show() {

        String urlStr = "https://samples.openweathermap.org/data/2.5/weather?q=London&mode=xml&appid=439d4b804bc8187953eb36d2a8c26a02";
        URL url;
        System.out.println(urlStr);
        try {
            url = new URL(urlStr);

            // 1. 웹주소에 연결: url에 커넥션 수립
            URLConnection connection = url.openConnection();

            // HTTP 프로토콜에 적합하게 HttpURLConnection로 캐스팅
            HttpURLConnection httpConnection = (HttpURLConnection) connection;

            // 2. 연결상태를 체크하여 정상일 때만 데이터 처리
            int resCode = httpConnection.getResponseCode();
            if (resCode == HttpURLConnection.HTTP_OK) {

                // 3. 자원에서 데이터를 읽을 수 있도록 InputStream을 얻는다.
                InputStream in = httpConnection.getInputStream();

                // 4. factory 객체 생성
                DocumentBuilderFactory factory =
                    DocumentBuilderFactory.newInstance();

                // DOM 객체 생성을 위한 builder 생성
                DocumentBuilder builder = factory.newDocumentBuilder();

                // 5. DOM 객체로 변환 : 요소 하나하나를 dom객체 형태로 받아옴
                Document dom = builder.parse( in );

                // 6. 각 객체를 읽음: document element를 읽어옴
                Element element = dom.getDocumentElement();

                //getElementsByTagName():  무조건 배열형태로 반환한다.

                Element cityTag = (Element) element.getElementsByTagName("city").item(0);
                String city = cityTag.getAttribute("name"); //속성값 읽어옴

                Element temperatureTag = (Element) element.getElementsByTagName("temperature").item(0);
                String temperature = temperatureTag.getAttribute("value");

                Element humidityTag = (Element) element.getElementsByTagName("humidity").item(0);
                String humidity = humidityTag.getAttribute("value") +
                    humidityTag.getAttribute("unit");

                Element cloudsTag = (Element) element.getElementsByTagName("clouds").item(0);
                String clouds = cloudsTag.getAttribute("name");

                WeatherInfo w = new WeatherInfo(city, temperature, humidity, clouds);
                System.out.println(w);
                tv.setText(w.toString());
            }

        } catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ParserConfigurationException e) {
            e.printStackTrace();
        } catch (SAXException e) {
            e.printStackTrace();
        }

    }

}
```

</details>

<br/>

> 결과화면<br/>

<img src="https://user-images.githubusercontent.com/62331803/91243583-473bf600-e785-11ea-8aa2-7cb08e16ee73.png" width="50%">

<br/>


## 예제3) 웹에서 json 파일 불러와서 사용하기


> 샘플 json 파일<br/>

![image](https://user-images.githubusercontent.com/62331803/91244236-d269bb80-e786-11ea-90e5-0e744fe1db34.png)

<br/>

> activity_main3.xml<br/>

activity_main2.xml 과 동일

<br/><br/>

> MainActivity3.java
<details>
<summary>코드보기</summary>

```java
package com.example.nettest;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.os.StrictMode;
import android.widget.TextView;
import android.widget.Toast;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.net.URLConnection;

public class MainActivity3 extends AppCompatActivity {
    private TextView tv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main3);
        tv = findViewById(R.id.tv);

        StrictMode.ThreadPolicy policy = new StrictMode.ThreadPolicy.Builder().permitAll().build();
        StrictMode.setThreadPolicy(policy);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            //For device above MarshMellow
            boolean permission = getInternetPermission();
            if (permission) {
                Toast.makeText(this, "obtain contact permission", Toast.LENGTH_SHORT).show();
            }
        } else {
            Toast.makeText(this, "failed to obtain contact permission", Toast.LENGTH_SHORT).show();
        }
        show();
    }

    public boolean getInternetPermission() {
        boolean hasPermission = (
            (ContextCompat.checkSelfPermission(this,
                Manifest.permission.INTERNET) == PackageManager.PERMISSION_GRANTED));
        if (!hasPermission) {
            ActivityCompat.requestPermissions(this, new String[] {
                Manifest.permission.INTERNET
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
                    Toast.makeText(this, "obtain internet permission", Toast.LENGTH_SHORT).show();
                }
            }
        }
    }

    public void show() {
        String urlStr = "https://samples.openweathermap.org/data/2.5/weather?q=London&appid=439d4b804bc8187953eb36d2a8c26a02";
        URL url;
        char[] data = new char[255];
        StringBuffer sb = new StringBuffer(); //모든 json 정보를 저장

        try {
            url = new URL(urlStr);
            // url에 커넥션 수립
            URLConnection connection = url.openConnection();
            // HTTP 프로토콜에 적합하게 HttpURLConnection로 캐스팅
            HttpURLConnection httpConnection = (HttpURLConnection) connection;
            // 연결상태를 체크하여 정상일 때만 데이터 처리
            int resCode = httpConnection.getResponseCode();
            if (resCode == HttpURLConnection.HTTP_OK) {
                // 자원에서 데이터를 읽을 수 있도록 InputStream을 얻는다.
                InputStream in = httpConnection.getInputStream();
                InputStreamReader isr = new InputStreamReader( in );
                while (isr.read(data) > 0) {
                    sb.append(data);
                    if ( in .available() < 255) { //쓰레기값 방지
                        for (int i = 0; i < data.length; i++) {
                            data[i] = ' ';
                        }
                    }
                }
                System.out.println(sb);
                isr.close();

                //<PARSING>
                //PARSING 방법 => 1) JSONObject(): 객체일 때 2)JSONArray(): 배열일 때
                JSONObject weatherInfo = new JSONObject(sb.toString());
                JSONObject coord = weatherInfo.getJSONObject("coord");
                JSONObject main = weatherInfo.getJSONObject("main");
                String name = weatherInfo.getString("name");
                Coord c = new Coord(coord.getDouble("lon"), coord.getDouble("lat"));
                Main m = new Main(main.getDouble("temp"), main.getDouble("pressure"), main.getDouble("humidity"));
                WeatherInfo2 w = new WeatherInfo2(c, m, name);
                tv.setText(w.toString());
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (JSONException e) {
            e.printStackTrace();
        }

    }

}
```

</details>

<br/>

> WeatherInfo2.java

<details>
<summary>코드보기</summary>

```java
package com.example.nettest;

/**
 * Created by usr on 2015-08-28.
 */
public class WeatherInfo2 {
    private Coord c;
    private Main m;
    private String name;

    public WeatherInfo2() {}

    public WeatherInfo2(Coord c, Main m, String name) {
        this.c = c;
        this.m = m;
        this.name = name;
    }

    public Coord getC() {
        return c;
    }

    public void setC(Coord c) {
        this.c = c;
    }

    public Main getM() {
        return m;
    }

    public void setM(Main m) {
        this.m = m;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "WeatherInfo2{" +
            "c=" + c +
            ", m=" + m +
            ", name='" + name + '\'' +
            '}';
    }
}
```

</details>
<br/>

> Coord.java | VO 클래스

<details>
<summary>코드보기</summary>

```java
package com.example.nettest;

public class Coord {
    private double lon;
    private double lat;

    public Coord() {}
    public Coord(double lon, double lat) {
        this.lon = lon;
        this.lat = lat;
    }

    public double getLon() {
        return lon;
    }

    public void setLon(double lon) {
        this.lon = lon;
    }

    public double getLat() {
        return lat;
    }

    public void setLat(double lat) {
        this.lat = lat;
    }

    @Override
    public String toString() {
        return "Coord{" +
            "lon='" + lon + '\'' +
            ", lat='" + lat + '\'' +
            '}';
    }
}
```

</details>

<br/>

> Main.java | VO 클래스

<details>
<summary>코드보기</summary>

```java
package com.example.nettest;

public class Main {
    private double temp;
    private double pressure;
    private double humidity;

    public Main() {}
    public Main(double temp, double pressure, double humidity) {
        this.temp = temp;
        this.pressure = pressure;
        this.humidity = humidity;
    }

    public double getTemp() {
        return temp;
    }

    public void setTemp(double temp) {
        this.temp = temp;
    }

    public double getPressure() {
        return pressure;
    }

    public void setPressure(double pressure) {
        this.pressure = pressure;
    }

    public double getHumidity() {
        return humidity;
    }

    public void setHumidity(double humidity) {
        this.humidity = humidity;
    }

    @Override
    public String toString() {
        return "Main{" +
            "temp=" + temp +
            ", pressure=" + pressure +
            ", humidity=" + humidity +
            '}';
    }
}
```

</details>


<br/>

> 결과화면<br/>

<img src="https://user-images.githubusercontent.com/62331803/91246102-04305180-e78a-11ea-8874-46e95c471185.png" width="50%">

<br/>

## 예제4) Socket 통신

> 안드로이드와 통신할 Server 시작

`Eclipse>>java1>>p0428>>EchoServer` <br/>

<img src="https://user-images.githubusercontent.com/62331803/91247154-95082c80-e78c-11ea-8a06-c8db4406f1e6.png" width="80%">

<details>
<summary> EchoServer.java 코드보기</summary>

```java
package p0428;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

//클라이언트 1명에 쓰레드 1개
class EchoServerTh extends Thread {
    private Socket socket;

    public EchoServerTh(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        // TODO Auto-generated method stub
        try {
            InputStream is = socket.getInputStream();
            InputStreamReader ir = new InputStreamReader(is);
            BufferedReader br = new BufferedReader(ir);
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
            while (true) {
                String str = br.readLine(); // 클라이언트가 보낸 메시지 한줄 읽음			
                System.out.println("클라이언트 메시지:" + str);
                out.println(str + "\n"); // 클라이언트가 보낸 메시지를 다시 클라이언트에게 보냄
                if (str.startsWith("/stop")) {
                    break;
                }
            }
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

public class EchoServer {

    public static void main(String[] args) {
        // TODO Auto-generated method stub
        try {
            ServerSocket ss = new ServerSocket(3333); // 서버소켓오픈. 파라메터는 포트번호
            System.out.println("서버시작");
            while (true) {
                Socket socket = ss.accept(); // 클라이언트 접속 기다리다 요청수락. 수락후 이 클라이언트와
                // 1:1통신할 소켓 반환
                System.out.println("클라이언트 접속");
                new EchoServerTh(socket).start();
            }

        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
```

</details>

<br/>

> activity_main4.xml <br/>

<img src="https://user-images.githubusercontent.com/62331803/91248164-faf5b380-e78e-11ea-877f-6a58c1a43749.png" width="70%">

<br/>

> MainActivity4.java

<details>
<summary>코드보기</summary>

```java
package com.example.nettest;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.os.StrictMode;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

public class MainActivity4 extends AppCompatActivity {
    private EditText msg;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main4);
        msg = findViewById(R.id.msg);

        StrictMode.ThreadPolicy policy = new StrictMode.ThreadPolicy.Builder().permitAll().build();
        StrictMode.setThreadPolicy(policy);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            //For device above MarshMellow
            boolean permission = getInternetPermission();
            if (permission) {
                Toast.makeText(this, "obtain contact permission", Toast.LENGTH_SHORT).show();
            }
        } else {
            Toast.makeText(this, "failed to obtain contact permission", Toast.LENGTH_SHORT).show();
        }
    }

    public boolean getInternetPermission() {
        boolean hasPermission = (
            (ContextCompat.checkSelfPermission(this,
                Manifest.permission.INTERNET) == PackageManager.PERMISSION_GRANTED));
        if (!hasPermission) {
            ActivityCompat.requestPermissions(this, new String[] {
                Manifest.permission.INTERNET
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
                    Toast.makeText(this, "obtain internet permission", Toast.LENGTH_SHORT).show();
                }
            }
        }
    }

    public void onSend(View view) {
        String m = msg.getText().toString();
        //        System.out.println("m:"+ m);
        MyThread th = new MyThread(m);
        th.start();
    }
}
```

</details>

<br/>

> MyThread.java

<details>
<summary>코드보기</summary>

```java
package com.example.nettest;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.net.InetAddress;
import java.net.Socket;
import java.net.UnknownHostException;

public class MyThread extends Thread {
    private String msg;

    public MyThread(String msg) {
        this.msg = msg;
    }

    @Override
    public void run() {
        try {
            System.out.println("new msg:" + msg);
            InetAddress serverAddr = InetAddress.getByName("192.168.22.110");
            Socket socket = new Socket(serverAddr, 3333);
            BufferedWriter out = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            out.write(msg + "\n");
            out.flush();
            msg = "received from " + in .readLine();
            System.out.println(msg);
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```

</details>

<br/>


> 결과화면<br/>

- 안드로이드(클라이언트)에서 전송<br/>
![image](https://user-images.githubusercontent.com/62331803/91252456-9b9ca100-e798-11ea-918c-6cf89cc9b665.png)

- 서버측 확인<br/>
![image](https://user-images.githubusercontent.com/62331803/91252479-a35c4580-e798-11ea-9394-26a319c0d6f4.png)

<br/>



# 2. WebApp 프로젝트 | 자바 웹어플리케이션과 연동(진행중)

<br/>

## 예제1) 로그인

### `1-1) 서버측 -  자바 웹`

> `web_and`  프로젝트 생성<br/>

<img src="https://user-images.githubusercontent.com/62331803/91259066-b5de7b00-e7a8-11ea-9246-5c20f06eb266.png" width="50%">

<br>

> build path<br/>

<img src="https://user-images.githubusercontent.com/62331803/91259176-f3db9f00-e7a8-11ea-8f3e-a99df0ba0538.png" width="60%">

<br/>


> servlet 생성 <br/>

`Java Resources>> src >> src >> login.java`

```java
import java.io.IOException;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/login")
public class login extends HttpServlet {
    private static final long serialVersionUID = 1 L;

    public login() {
        // TODO Auto-generated constructor stub
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // TODO Auto-generated method stub
        String id = request.getParameter("id");
        String pwd = request.getParameter("pwd");
        boolean flag = false;
        if (id.equals("aaa") && pwd.equals("111")) {
            flag = true;
        }
        request.setAttribute("flag", flag);
        RequestDispatcher rd = request.getRequestDispatcher("views/login.jsp");
        rd.forward(request, response);
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // TODO Auto-generated method stub
        doGet(request, response);
    }

}
```

<br/>

> 뷰 페이지 생성 <br/>

`WebContent>>views>>login.jsp`

```jsp
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
{"flag":${flag}}
```

<br/>

### `1-2) 클라이언트측 - 안드로이드`


> build.gradle(Module:app) 추가<br/>

뷰모델 사용을 위한 생명주기 extensions 설정

```gradle
def dependencies{
...
		implementation 'androidx.lifecycle:lifecycle-extensions:2.2.0'
}
```

<br/>

> manifest.xml<br/>

안드로이는 웹 연동시 https를 권장.<br/>
http방식을 허용하기 위한 설정 추가
```xml
<application  
 ...
  android:usesCleartextTraffic="true">
```

<br/>

> RequestHttpURLConnection.java

<details>
<summary>코드보기</summary>

```java
package com.example.webapp;

import android.content.ContentValues;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.util.Map;

//HTTP 작업 구현해놓은 클래스
public class RequestHttpURLConnection {
    public String request(String _url, ContentValues _params) {

        // HttpURLConnection 참조 변수.
        HttpURLConnection urlConn = null;
        // URL 뒤에 붙여서 보낼 파라미터.
        StringBuffer sbParams = new StringBuffer();

        /**
         * 1. StringBuffer에 파라미터 연결
         * */
        // 보낼 데이터가 없으면 파라미터를 비운다.
        if (_params == null)
            sbParams.append("");
        // 보낼 데이터가 있으면 파라미터를 채운다.
        else {
            // 파라미터가 2개 이상이면 파라미터 연결에 &가 필요하므로 스위칭할 변수 생성.
            boolean isAnd = false;
            // 파라미터 키와 값.
            String key;
            String value;

            for (Map.Entry < String, Object > parameter: _params.valueSet()) {
                key = parameter.getKey();
                value = parameter.getValue().toString();

                // 파라미터가 두개 이상일때, 파라미터 사이에 &를 붙인다.
                if (isAnd)
                    sbParams.append("&");

                sbParams.append(key).append("=").append(value);

                // 파라미터가 2개 이상이면 isAnd를 true로 바꾸고 다음 루프부터 &를 붙인다.
                if (!isAnd)
                    if (_params.size() >= 2)
                        isAnd = true;
            }
        }

        /**
         * 2. HttpURLConnection을 통해 web의 데이터를 가져온다.
         * */
        try {
            URL url = new URL(_url);
            urlConn = (HttpURLConnection) url.openConnection();

            // [2-1]. urlConn 설정.
            urlConn.setRequestMethod("POST"); // URL 요청에 대한 메소드 설정 : POST.
            urlConn.setRequestProperty("Accept-Charset", "UTF-8"); // Accept-Charset 설정.
            urlConn.setRequestProperty("Context_Type", "application/x-www-form-urlencoded;cahrset=UTF-8");

            // [2-2]. parameter 전달 및 데이터 읽어오기.
            String strParams = sbParams.toString(); //sbParams에 정리한 파라미터들을 스트링으로 저장. 예)id=id1&pw=123;
            OutputStream os = urlConn.getOutputStream();
            os.write(strParams.getBytes("UTF-8")); // 출력 스트림에 출력.
            os.flush(); // 출력 스트림을 플러시(비운다)하고 버퍼링 된 모든 출력 바이트를 강제 실행.
            os.close(); // 출력 스트림을 닫고 모든 시스템 자원을 해제.
            System.out.println("param:" + strParams);
            // [2-3]. 연결 요청 확인.
            // 실패 시 null을 리턴하고 메서드를 종료.
            if (urlConn.getResponseCode() != HttpURLConnection.HTTP_OK) {
                System.out.println("연결실패");
                return null;
            }

            // [2-4]. 읽어온 결과물 리턴.
            // 요청한 URL의 출력물을 BufferedReader로 받는다.
            BufferedReader reader = new BufferedReader(new InputStreamReader(urlConn.getInputStream(), "UTF-8"));

            // 출력물의 라인과 그 합에 대한 변수.
            String line;
            String page = "";

            // 라인을 받아와 합친다.
            while ((line = reader.readLine()) != null) {
                page += line;
            }
            System.out.println("result:" + page);
            return page;

        } catch (MalformedURLException e) { // for URL.
            e.printStackTrace();
        } catch (IOException e) { // for openConnection().
            e.printStackTrace();
        } finally {
            if (urlConn != null)
                urlConn.disconnect();
        }

        return null;

    }
}
```


</details>

<br/>


> webViewModel.java 

<details>
<summary>코드보기</summary>

```java
package com.example.webapp;

import android.content.ContentValues;

import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.ViewModel;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class WebViewModel extends ViewModel {
    private MutableLiveData < String > res;
    private ExecutorService executorService;
    private RequestHttpURLConnection conn;

    public void setRes(MutableLiveData < String > res) {
        this.res = res;
    }

    public WebViewModel() {
        executorService = Executors.newSingleThreadExecutor();
        conn = new RequestHttpURLConnection();
    }

    public void req(String url, ContentValues cv) {
        String r = conn.request(url, cv);
        System.out.println(r);
        res.postValue(r);
    }

}
```


</details>

<br/>

> activity_main.xml<br/>

<img src="https://user-images.githubusercontent.com/62331803/91257422-c42a9800-e7a4-11ea-914f-e8952bb1b8da.png" width="70%">

<br/>

> MainActivity.java
<details>
<summary>코드보기</summary>

```java
package com.example.webapp;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Intent;
import android.os.Bundle;
import android.os.StrictMode;
import android.view.View;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        StrictMode.ThreadPolicy policy = new StrictMode.ThreadPolicy.Builder().permitAll().build();
        StrictMode.setThreadPolicy(policy);

    }

    public void onLogin(View view) {
        Intent intent = new Intent(this, LoginActivity.class);
        startActivity(intent);
    }
}
```

</details>

<br/>


> activity_login.xml<br/>

<img src="https://user-images.githubusercontent.com/62331803/91258699-f38ed400-e7a7-11ea-8bf0-04662aeaf2a0.png" width="70%">

<br/>

> LoginActivity.java

<details>
<summary>코드보기</summary>

```java
package com.example.webapp;  
  
import androidx.annotation.NonNull;  
import androidx.appcompat.app.AppCompatActivity;  
import androidx.core.app.ActivityCompat;  
import androidx.core.content.ContextCompat;  
import androidx.lifecycle.MutableLiveData;  
import androidx.lifecycle.Observer;  
import androidx.lifecycle.ViewModelProvider;  
  
import android.Manifest;  
import android.content.ContentValues;  
import android.content.pm.PackageManager;  
import android.os.Build;  
import android.os.Bundle;  
import android.os.StrictMode;  
import android.view.View;  
import android.widget.EditText;  
import android.widget.Toast;  
  
import org.json.JSONException;  
import org.json.JSONObject;  
  
public class LoginActivity extends AppCompatActivity {  
    private EditText id_et;  
 private EditText pwd_et;  
 private WebViewModel model;  
 private MutableLiveData<String> res;  
  
  @Override  
  protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
  setContentView(R.layout.activity_login2);  
  id_et = findViewById(R.id.id);  
  pwd_et = findViewById(R.id.pwd);  
  res = new MutableLiveData<>();  
  model = new ViewModelProvider(this).get(WebViewModel.class);  
  model.setRes(res);  
  
  res.observe(this, new Observer<String>() {  
            @Override  
  public void onChanged(String s) {  
                String str = "";  
 try {  
                    JSONObject obj = new JSONObject(s);  
 if(obj.getBoolean("flag")){  
                        str = "로그인 성공";  
  } else{  
                        str = "로그인 실패";  
  }  
                    Toast.makeText(LoginActivity.this, str ,Toast.LENGTH_SHORT).show();  
  } catch (JSONException e) {  
                    e.printStackTrace();  
  }  
            }  
        });  
  }  
  
    public void onLogin(View view){  
        String id = id_et.getText().toString();  
//        System.out.println(id);  
  String pwd = pwd_et.getText().toString();  
//        System.out.println(pwd);  
  ContentValues cv = new ContentValues();  
  cv.put("id",id);  
  cv.put("pwd",pwd);  
  System.out.println("안보냄");  
  model.req("http://192.168.22.110:8989/web_and/login",cv);  
  System.out.println("보냄");  
  }  
}
```

</details>

<br/>

> 결과화면 <br/>

1. 로그인 <br/>
<img src="https://user-images.githubusercontent.com/62331803/91266149-086d6680-e7ac-11ea-8aeb-2aa6ba2c2a70.png" width="50%">


2. 뷰 페이지 확인<br/>
<img src="https://user-images.githubusercontent.com/62331803/91259675-e8d53e80-e7a9-11ea-8663-1eb8292d83d2.png" width="80%">




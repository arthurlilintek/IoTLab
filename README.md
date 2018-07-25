# IoTLab


package com.lilintek.lab;

import android.graphics.Bitmap;
import android.graphics.Point;
import android.graphics.drawable.BitmapDrawable;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.DisplayMetrics;
import android.util.Log;
import android.view.Display;
import android.view.MotionEvent;
import android.view.View;
import android.view.Menu;
import android.view.MenuItem;
import android.webkit.WebView;
import android.widget.ImageView;
import android.widget.TextView;

import com.google.firebase.database.ChildEventListener;
import com.google.firebase.database.DataSnapshot;
import com.google.firebase.database.DatabaseError;
import com.google.firebase.database.DatabaseReference;
import com.google.firebase.database.FirebaseDatabase;

import java.util.Date;


public class MainActivity extends AppCompatActivity {
    java.util.ArrayDeque<String> tq=new java.util.ArrayDeque<>();
    java.util.ArrayDeque<String> hq=new java.util.ArrayDeque<>();
    java.util.ArrayDeque<String> dq=new java.util.ArrayDeque<>();
    java.util.Date prev=new java.util.Date();
    final int points=10;
    DisplayMetrics displaymetrics = new DisplayMetrics();
    float density;
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        density= getResources().getDisplayMetrics().density;
        FirebaseDatabase database = FirebaseDatabase.getInstance();
        final DatabaseReference rgbRef= database.getReference("/rgbdata/pixel");
        final DatabaseReference logRef= database.getReference("/logs");
        final ImageView ilv=(ImageView) findViewById(R.id.colorp);
        final Bitmap bitmap = ((BitmapDrawable)ilv.getDrawable()).getBitmap();
        ilv.setOnTouchListener(new View.OnTouchListener(){
            @Override
            public boolean onTouch(View v, MotionEvent event){
                int x = (int)(event.getX());
                int y = (int)(event.getY());
                int pixel = bitmap.getPixel(x,y);
                //               int A = (pixel >> 24) & 0xff; // or color >>> 24
                //               int R = (pixel >> 16) & 0xff;
                //               int G = (pixel >>  8) & 0xff;
                //               int B = (pixel      ) & 0xff;
                //               Log.d("Color:",R+","+G+","+B);
//                rgbRef.setValue(Color.red(pixel)*65536+Color.green(pixel)*256+Color.blue(pixel));
                rgbRef.setValue(pixel&0xffffff);
//                Log.d("Color:",Color.alpha(pixel)+","+Color.red(pixel)+","+Color.green(pixel)+","+Color.blue(pixel));
//                Log.d("xy:",x+","+y);
                return false;
            }
        });

        ChildEventListener childEventListener = new ChildEventListener() {
            @Override
            public void onChildAdded(DataSnapshot dataSnapshot, String previousChildName) {
                java.util.Date now=new java.util.Date();
                String key=dataSnapshot.getKey();
                Log.d("Logs", "onChildAdded:" + key);
                String data=dataSnapshot.getValue().toString();
                String datas[]=data.split(",");
                if(datas==null||datas.length<3||datas[1]==null||datas[2]==null) return;
                Log.d("Time", "Date Time:" + new Date(Long.parseLong(key)*1000).toGMTString());
                dq.add(String.format("%1$tH:%1$tM:%1$tS",new Date(Long.parseLong(key)*1000-8*60*60*1000)));
                if(dq.size()>points)  dq.removeFirst();

                TextView lt=(TextView) findViewById(R.id.t);
                lt.setText("溫度(C): " +datas[1]);
                TextView lh=(TextView) findViewById(R.id.h);
                lh.setText("溼度(%): " +datas[2]);

                tq.add(String.format("%.2f",Double.parseDouble(datas[1])+20));
                if(tq.size()>points)  tq.removeFirst();

                hq.add(String.format("%.2f",(Double.parseDouble(datas[2])-30)*10/7));
                if(hq.size()>points)hq.removeFirst();

                if (now.getTime()-prev.getTime()>3000) {
                    Display display = getWindowManager().getDefaultDisplay();
                    Point size = new Point();
                    display.getRealSize(size);
                    int width =380;// Math.min(600, (int) (size.x / density));
                    int height = 180;//Math.min(500, (int) ((size.y - ilv.getHeight()) / density) - 16);

                    Log.d("xy:", size.x + "," + size.y);
                    String GraphURL = "http://chart.googleapis.com/chart?chs=" + width + "x" + height + "&chxt=x,y,y,r,r&chxp=2,90|4,90&chxr=1,-20,80|3,30,100&cht=lc&chco=FF0000,3366CC&chdl=Temprature|Humidity&chdlp=t&chg=0,-1&chtt=Temprature%20and%20Humidity%20Change&chm=o,0066FF,0,-1,3|d,ff0000,1,-1,3&chd=t:";
                    StringBuffer d = new StringBuffer();
                    for (String s : tq) {
                        d.append(s + ",");
                    }
                    d.replace(d.length() - 1, d.length(), "|");
                    for (String s : hq) {
                        d.append(s + ",");
                    }
                    d.deleteCharAt(d.length()-1);
                    d.append("&chxs=0,,8&chxl=2:|C|4:|%|0:|");
                    for (String s : dq) {
                        d.append(s + "|");
                    }
                    d.deleteCharAt(d.length()-1);
                    GraphURL+=d;

                    Log.d("GraphURL :", GraphURL);
                    WebView lwv = (WebView) findViewById(R.id.wv);
                    lwv.loadUrl(GraphURL);
                }
                prev=now;
                //Log.d("now :", now.toString());
            }

            @Override
            public void onChildChanged(DataSnapshot dataSnapshot, String s) {
            }
            @Override
            public void onChildRemoved(DataSnapshot dataSnapshot) {
            }
            @Override
            public void onChildMoved(DataSnapshot dataSnapshot, String s) {
            }
            @Override
            public void onCancelled(DatabaseError databaseError) {
            }
        };
        logRef.addChildEventListener(childEventListener);

        //WebView lwv = (WebView)findViewById(R.id.wv);
       // lwv.loadUrl("file:///android_asset/linechart.html");
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();
      if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }

}

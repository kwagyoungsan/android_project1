### LoginRequest.java
```
package com.example.project2;

import android.util.Log;

import com.android.volley.Response;
import com.android.volley.VolleyError;
import com.android.volley.toolbox.StringRequest;

import java.util.HashMap;
import java.util.Map;

public class LoginRequest extends StringRequest {
    final static private String URL = "http://super9609.dothome.co.kr/AccountManagement/Login.php";
    private Map<String, String> parameters;
    public LoginRequest(String userID, String userPassword, Response.Listener<String> listener) {
        super(Method.POST, URL, listener, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                Log.e("RegisterRequest", error.getMessage());
            }
        });
        parameters = new HashMap<>();
        parameters.put("userID", userID);
        parameters.put("userPassword", userPassword);
    }
    @Override
    public Map<String, String> getParams() {
        return parameters;
    }
}
```

### MainActivity.java
```
package com.example.project2;

import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentManager;
import androidx.fragment.app.FragmentTransaction;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import android.content.Intent;
import android.content.SharedPreferences;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import com.android.volley.RequestQueue;
import com.android.volley.Response;
import com.android.volley.toolbox.Volley;

import org.json.JSONException;
import org.json.JSONObject;

import java.io.Serializable;
import java.util.ArrayList;

import static android.os.Build.ID;

public class MainActivity extends AppCompatActivity {
    EditText IDIn, PasswordIn;
    String LoginID, LoginPassword;

    SQLiteDatabase db;
    Bundle bundle;
//    DBHelper mHelper;

    final static int REQCODE_ACTEDIT = 1001;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
//        mHelper = new DBHelper(this);
        IDIn = (EditText) findViewById(R.id.idid);
        PasswordIn = (EditText) findViewById(R.id.pwpw);
        Intent intent = getIntent();
        LoginID = intent.getStringExtra("IDIn");
        LoginPassword = intent.getStringExtra("PasswordIn");

//        bundle.putString("LoginID",ID);

        SharedPreferences pref = getSharedPreferences("PrefTest", MODE_PRIVATE);
        String loginid = pref.getString("LoginID", "");
        if (!loginid.equals(""))
            IDIn.setText(loginid);
        String loginpw = pref.getString("LoginPassword", "");
        if (!loginpw.equals(""))
            PasswordIn.setText(loginpw);

    }

    public void LoginButton(View v) {

//        int check = 0;

//        db = mHelper.getReadableDatabase();
//        Cursor cursor;
//        cursor = db.rawQuery("SELECT name, id, password, department FROM person", null);
//
//        while (cursor.moveToNext()) {
//            String name = cursor.getString(0);
//            String id = cursor.getString(1);
//            String password = cursor.getString(2);
//            String department = cursor.getString(3);
//
//            if (IDIn.getText().toString().equals(id) &&
//                    PasswordIn.getText().toString().equals(password)) {
//                Intent intent = new Intent(this, Navigation.class);
//                intent.putExtra("personid", id);
//                intent.putExtra("personname", name);
//                startActivityForResult(intent, REQCODE_ACTEDIT);
//                check = 1;
//            }
//
//        }
//        cursor.close();
//        db.close();
//        if (check != 1) {
//            Toast.makeText(this.getApplicationContext(), "잘못 입력하였습니다.",
//                    Toast.LENGTH_SHORT).show();
//        }

        Intent intent = new Intent(this, Navigation.class);



        String userID = IDIn.getText().toString();
        String userPassword = PasswordIn.getText().toString();



//        Toast.makeText(this.getApplicationContext(), "메인 : "+userID,  Toast.LENGTH_SHORT).show();

        Response.Listener<String> responseListener = new Response.Listener<String>() {
            @Override
            public void onResponse(String response) {
                try {
                    JSONObject jsonResponse = new JSONObject(response);
                    boolean isSuccess = jsonResponse.getBoolean("success");
                    if (isSuccess) {
                        String userID = jsonResponse.getString("userID");
                        String userPassword = jsonResponse.getString("userPassword");
                        Intent intent = new Intent(MainActivity.this, Navigation.class);
                        intent.putExtra("userID", userID);
                        intent.putExtra("userPassword", userPassword);
                        startActivity(intent);
                    } else {
                        AlertDialog.Builder builder = new AlertDialog.Builder(MainActivity.this);
                        builder.setMessage("Login Failed.").setNegativeButton("Retry", null).create().show();
                    }
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }
        };
        LoginRequest loginRequest = new LoginRequest(userID, userPassword, responseListener);
        RequestQueue requestQueue = Volley.newRequestQueue(MainActivity.this);
        requestQueue.add(loginRequest);

    }

    public void SignupButton(View v) {
        Intent intent = new Intent(this, SubActivity.class);
        startActivity(intent);
    }

    public void checkbox(View v) {
        SharedPreferences pref = getSharedPreferences("PrefTest", MODE_PRIVATE);
        SharedPreferences.Editor edit = pref.edit();
        String ID = IDIn.getText().toString();
        String PW = PasswordIn.getText().toString();

        edit.putString("LoginID", ID);
        edit.putString("LoginPassword", PW);
        edit.commit();
    }
}
```

### MainFragment.java
```
package com.example.project2;

import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ListView;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentManager;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.android.volley.RequestQueue;
import com.android.volley.Response;
import com.android.volley.toolbox.Volley;

import org.json.JSONException;
import org.json.JSONObject;

import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;

public class MainFragment extends Fragment {

    EditText receiver, title, content;
    TextView date;
//    DBHelper mHelper;
    SQLiteDatabase dbread;
    Message message;
    ArrayList<Message> messages;
    ArrayAdapter<Message> adapter;
    ListView list;
    Button cancel, send, plus;
    Context context;
    Bundle bundle;
    Cursor cursor;
    String GETID;

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        ViewGroup rootView = (ViewGroup) inflater.inflate(R.layout.newmessage, container, false);

        receiver = rootView.findViewById(R.id.Receiver);
        title = rootView.findViewById(R.id.Title);
        content = rootView.findViewById(R.id.Content);
        date = rootView.findViewById(R.id.date);

        cancel = rootView.findViewById(R.id.cancel);
        send = rootView.findViewById(R.id.send);
        plus = rootView.findViewById(R.id.plus);

        messages = new ArrayList<Message>();
        context = container.getContext();

//        mHelper = new DBHelper(getActivity());

        bundle = getArguments();

        GETID = bundle.getString("PersonID");
        Toast.makeText(getActivity(), GETID, Toast.LENGTH_SHORT).show();
        long now = System.currentTimeMillis();
        Date date = new Date(now);
        SimpleDateFormat mFormat = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
        final String time = mFormat.format(date);


        send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
//                dbread = mHelper.getReadableDatabase();

//                cursor = dbread.rawQuery("SELECT receiver , title, content ,send,date FROM message", null);
//
//                long now = System.currentTimeMillis();
//                Date mDate = new Date(now);
//                SimpleDateFormat simpleDate = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
//                String getTime = simpleDate.format(mDate);
//
//                message = new Message(receiver.getText().toString(), title.getText().toString(), content.getText().toString(), bundle.getString("PersonID"),getTime);
////                mHelper.addMessage(message);
//                Toast.makeText(context.getApplicationContext(), "전송 완료", Toast.LENGTH_SHORT).show();
//                FragmentManager fragmentManager = getActivity().getSupportFragmentManager();
//                fragmentManager.beginTransaction().remove(MainFragment.this).commit();
//                fragmentManager.popBackStack();
                String Receiver = receiver.getText().toString();
                String Title = title.getText().toString();
                String Content = content.getText().toString();
//                String Send = content.getText().toString();
//                String Date =mDepartment.getSelectedItem().toString();

                message = new Message(receiver.getText().toString(), title.getText().toString(), content.getText().toString(), GETID, time);

                Response.Listener<String> responseListener = new Response.Listener<String>() {
                    @Override
                    public void onResponse(String response) {
                        try {
                            JSONObject jsonResponse = new JSONObject(response);
                            boolean isSuccess = jsonResponse.getBoolean("success");
                            if (isSuccess) {
                                AlertDialog.Builder builder = new AlertDialog.Builder(context);
                                builder.setMessage("Send Success.");
                                builder.setPositiveButton("OK", new DialogInterface.OnClickListener() {
                                    @Override
                                    public void onClick(DialogInterface dialog, int which) {
                                        dialog.dismiss();
                                        Intent intent = new Intent(context, MainActivity.class);
                                        startActivity(intent);
                                    }
                                });
                                builder.create().show();
                            } else {
                                AlertDialog.Builder builder = new AlertDialog.Builder(context);
                                builder.setMessage("Send Failed.").setNegativeButton("Retry", null).create()
                                        .show();
                            }
                        } catch (JSONException e) {
                            e.printStackTrace();
                        }
                    }
                };

                MessageRequest messageRequest
                        = new MessageRequest(Receiver, Title, Content, GETID, time, responseListener);
                RequestQueue requestQueue = Volley.newRequestQueue(context);
                requestQueue.add(messageRequest);
            }
        });

        cancel.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                FragmentManager fragmentManager = getActivity().getSupportFragmentManager();
                fragmentManager.beginTransaction().remove(MainFragment.this).commit();
                fragmentManager.popBackStack();
            }
        });


        plus.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {




//                FragmentManager fragmentManager = getActivity().getSupportFragmentManager();
//                fragmentManager.beginTransaction().remove(MainFragment.this).commit();
//                fragmentManager.popBackStack();
            }
        });
        return rootView;
    }
}
```

### MediaScanner.java
```
package com.example.project2;

import android.content.Context;
import android.media.MediaScannerConnection;
import android.net.Uri;
import android.util.Log;

public class MediaScanner {
    private Context mContext;
    private String mPath;
    private MediaScannerConnection mMediaScanner;
    private MediaScannerConnection.MediaScannerConnectionClient mMediaScannerClient;

    public static MediaScanner newInstance(Context context) {
        return new MediaScanner(context);
    }

    private MediaScanner(Context context) {
        mContext = context;
    }

    public void mediaScanning(final String path) {
        if (mMediaScanner == null) {
            mMediaScannerClient = new MediaScannerConnection.MediaScannerConnectionClient() {
                @Override
                public void onMediaScannerConnected() {
                    mMediaScanner.scanFile(mPath, null);
                }

                @Override
                public void onScanCompleted(String path, Uri uri) {
                    Log.d("Success : ", "MediaScan Complete!");
                    mMediaScanner.disconnect();
                }
            };
            mMediaScanner = new MediaScannerConnection(mContext, mMediaScannerClient);
        }
        mPath = path;
        mMediaScanner.connect();
    }
}
```

### Message.java
```

package com.example.project2;

import android.widget.TextView;

import java.text.SimpleDateFormat;
import java.util.Date;

class Message {
    private int icon;
    private String receiver;
    private String title;
    private String content;
    private String date;
    private String send;


    public Message( String Send, String Content, String Date) {
//        icon = Icon;
        send = Send;
        content = Content;
        date = Date;
    }

    public Message(String Receiver, String Title, String Content, String Send, String Date) {
        receiver = Receiver;
        title = Title;
        content = Content;
        send = Send;
        date = Date;
    }



    public int getIcon() {
        return icon;
    }

    public String getReceiver() {
        return receiver;
    }

    public String getTitle() {
        return title;
    }

    public String getContent() {
        return content;
    }

    public String getDate() {
        return date;
    }

    public String getSend() {
        return send;
    }
}
```
### MessageAdapter.java
```
package com.example.project2;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import java.util.ArrayList;

public class MessageAdapter extends RecyclerView.Adapter<MessageAdapter.MyViewHolder> {
    ArrayList<Message> messages;


    public MessageAdapter(ArrayList<Message> messages) {
        this.messages = messages;
    }

    public interface OnItemClickListener {
        void onItemClick(View v, int position) ;
    }

    private static OnItemClickListener mListener = null ;

    public void setOnItemClickListener(OnItemClickListener listener) {
        this.mListener = listener ;
    }



    @NonNull
    @Override
    public MyViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        LayoutInflater inflater = LayoutInflater.from(parent.getContext());
        View itemView = inflater.inflate(R.layout.listmessage, parent, false);
        return new MyViewHolder(itemView);
    }

    @Override
    public void onBindViewHolder(@NonNull MyViewHolder holder, int position) {
        Message message = messages.get(position);
        holder.setMessage(message);
    }

    @Override
    public int getItemCount() {
        return messages.size();
    }

    static class MyViewHolder extends RecyclerView.ViewHolder {
        ImageView icon;

        TextView title, content, date;


        public MyViewHolder(View itemView) {
            super(itemView);
            icon = itemView.findViewById(R.id.icon);
            title = itemView.findViewById(R.id.send);
            content = itemView.findViewById(R.id.content);
            date = itemView.findViewById(R.id.date);

            itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    int pos = getAdapterPosition() ;
                    if (pos != RecyclerView.NO_POSITION) {
                        if (mListener != null) {
                            mListener.onItemClick(v, pos) ;
                        }
                    }
                }
            });
        }

        public void setMessage(Message message) {
//            icon.setImageResource(message.getIcon());
            title.setText(message.getTitle());
            content.setText(message.getContent());
            date.setText(message.getDate());
        }
    }
}

```
### MessagePage.java
```
package com.example.project2;

import android.content.Intent;
import android.os.Bundle;
import android.widget.EditText;

import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import java.util.ArrayList;


public class MessagePage extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.messagepage);




    }
}
```
### MessageRequest.java
```
package com.example.project2;

import android.util.Log;

import com.android.volley.Response;
import com.android.volley.VolleyError;
import com.android.volley.toolbox.StringRequest;

import java.util.HashMap;
import java.util.Map;

public class MessageRequest extends StringRequest {
    final static private String URL = "http://super9609.dothome.co.kr/AccountManagement/Message.php";
    private Map<String, String> parameters;
    public MessageRequest(String RECEIVER, String TITLE, String CONTENT, String SEND, String DATE, Response.Listener<String> listener) {
        super(Method.POST, URL, listener, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                Log.e("MessageRequest", error.getMessage());
            }
        });
        parameters = new HashMap<>();
        parameters.put("RECEIVER", RECEIVER);
        parameters.put("TITLE", TITLE);
        parameters.put("CONTENT", CONTENT);
        parameters.put("SEND", SEND);
        parameters.put("DATE", DATE);
    }
    @Override
    public Map<String, String> getParams() {
        return parameters;
    }
}
```
### Navigation.java
```
package com.example.project2;

import android.content.Intent;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.view.MenuItem;
import android.view.View;
import android.view.Menu;
import android.widget.LinearLayout;
import android.widget.Toast;

import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.google.android.material.snackbar.Snackbar;
import com.google.android.material.navigation.NavigationView;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.ActionBarDrawerToggle;
import androidx.core.view.GravityCompat;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentManager;
import androidx.fragment.app.FragmentTransaction;
import androidx.navigation.NavController;
import androidx.navigation.ui.AppBarConfiguration;
import androidx.navigation.ui.NavigationUI;
import androidx.drawerlayout.widget.DrawerLayout;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.Toolbar;

import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;

public class Navigation extends AppCompatActivity implements NavigationView.OnNavigationItemSelectedListener {

    LinearLayout WriteContainer, MessageContainer, SettingContainter, fragmentContainer, SendContainer;
    String PersonID, PersonName;
    Bundle bundle;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_navigation);

        Toolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        getSupportActionBar().setTitle("받은 메시지함");

        DrawerLayout drawer = findViewById(R.id.drawer_layout);
        ActionBarDrawerToggle toggle = new ActionBarDrawerToggle(this, drawer, toolbar, R.string.navigation_drawer_open, R.string.navigation_drawer_close);
        drawer.addDrawerListener(toggle);
        toggle.syncState();

        NavigationView navigationView = findViewById(R.id.nav_view);
        navigationView.setNavigationItemSelectedListener(this);

        WriteContainer = findViewById(R.id.WriteContainer);
        MessageContainer = findViewById(R.id.MessageContainer);
        SendContainer = findViewById(R.id.SendContainer);
        SettingContainter = findViewById(R.id.SettingContainer);
        fragmentContainer = findViewById(R.id.fragmentContainer);

        Intent intent = getIntent();

        PersonID = intent.getStringExtra("userID");
        PersonName = intent.getStringExtra("personname");

        Toast.makeText(this.getApplicationContext(), "네비게이션 : " + PersonID, Toast.LENGTH_SHORT).show();

        bundle = new Bundle(2);
        bundle.putString("PersonID", PersonID);
        bundle.putString("PersonName", PersonName);
//        Toast.makeText(this.getApplicationContext(), PersonName, Toast.LENGTH_SHORT).show();
    }


    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.navigation, menu);
        return true;
    }

    @Override
    public boolean onNavigationItemSelected(@NonNull MenuItem item) {
        int id = item.getItemId();

        FragmentManager fragmentManager = getSupportFragmentManager();
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

        if (id == R.id.write) {
            WriteContainer.setVisibility(View.GONE);
            MainFragment mainfragment = new MainFragment();
            getSupportActionBar().setTitle("메시지 작성");
            mainfragment.setArguments(bundle);
            fragmentTransaction.replace(R.id.fragmentContainer, mainfragment);
            fragmentTransaction.commit();
        } else if (id == R.id.mail) {
            WriteContainer.setVisibility(View.GONE);
            SubFragment subfragment = new SubFragment();
            getSupportActionBar().setTitle("받은 메시지함");
            subfragment.setArguments(bundle);
            fragmentTransaction.replace(R.id.fragmentContainer, subfragment);
            fragmentTransaction.commit();
        }
//        else if (id == R.id.sendmail) {
//            WriteContainer.setVisibility(View.GONE);
//            SendFragment sendfragment = new SendFragment();
//            getSupportActionBar().setTitle("보낸 메시지함");
//            sendfragment.setArguments(bundle);
//            fragmentTransaction.replace(R.id.fragmentContainer, sendfragment);
//            fragmentTransaction.commit();
//        }
        else if (id == R.id.setting) {
            WriteContainer.setVisibility(View.GONE);
            SettingFragment settingfragment = new SettingFragment();
            getSupportActionBar().setTitle("개인 정보 설정");
            settingfragment.setArguments(bundle);
            fragmentTransaction.replace(R.id.fragmentContainer, settingfragment);
            fragmentTransaction.commit();
        } else if (id == R.id.logout) {
            System.exit(0);
        }

        DrawerLayout drawer = (DrawerLayout) findViewById(R.id.drawer_layout);
        drawer.closeDrawer(GravityCompat.START);
        return true;
    }

    public void replaceFragment(Fragment fragment) {
        FragmentManager fragmentManager = getSupportFragmentManager();
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
        fragmentTransaction.replace(R.id.fragmentContainer, fragment).commit();
    }
}

```
### Person.java
```
package com.example.project2;

import java.io.Serializable;

public class Person implements Serializable {
    private String name, id, password, department;

    public Person(String n, String i, String p, String d) {
        name = n;
        id = i;
        password = p;
        department = d;
    }

    public String getName() {
        return name;
    }

    public String getID() {
        return id;
    }

    public String getPassword() {
        return password;
    }

    public String getDepartment() {
        return department;
    }
}

```
### ReadMessage.java
```
package com.example.project2;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

public class ReadMessage extends Fragment {
    @Nullable
    Bundle bundle;
    TextView receiver, send, title, date, content;

    String time;
//    DBHelper mHelper;
    SQLiteDatabase db;
    ContentValues row;
    Context context;
    Button join;


    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        ViewGroup rootView = (ViewGroup) inflater.inflate(R.layout.readmessage, container, false);

        receiver = rootView.findViewById(R.id.Receiver);
        send = rootView.findViewById(R.id.Send);
        title = rootView.findViewById(R.id.Title);
        date = rootView.findViewById(R.id.Date);
        content = rootView.findViewById(R.id.Content);

        bundle = getArguments();
        time = bundle.getString("date");

//        mHelper = new DBHelper(getActivity());
//        db = mHelper.getReadableDatabase();

        Cursor cursor;
        cursor = db.rawQuery("SELECT receiver , send, title , date, content FROM message", null);
        while (cursor.moveToNext()) {
            String Receiver = cursor.getString(0);
            String Send = cursor.getString(1);
            String Title = cursor.getString(2);
            String Date = cursor.getString(3);
            String Content = cursor.getString(4);

            if(Date.equals(time)){
                receiver.setText(Receiver);
                send.setText(Send);
                title.setText(Title);
                date.setText(Date);
                content.setText(Content);

            }
        }
        Toast.makeText(getActivity(), time, Toast.LENGTH_SHORT).show();


        return rootView;
    }

}

```
### RegisterRequest.java
```
package com.example.project2;

import android.util.Log;

import com.android.volley.Request;
import com.android.volley.Response;
import com.android.volley.VolleyError;
import com.android.volley.toolbox.StringRequest;

import java.util.HashMap;
import java.util.Map;

public class RegisterRequest extends StringRequest {
    final static private String URL = "http://super9609.dothome.co.kr/AccountManagement/Register.php";
    private Map<String, String> parameters;
    public RegisterRequest(String userID, String userPassword, String userName,
                           String userDepartment, Response.Listener<String> listener) {
        super(Request.Method.POST, URL, listener, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                Log.e("RegisterRequest", error.getMessage());
            }
        });
        parameters = new HashMap<>();
        parameters.put("userID", userID);
        parameters.put("userPassword", userPassword);
        parameters.put("userName", userName);
        parameters.put("userDepartment", userDepartment);
    }
    @Override
    public Map<String, String> getParams() {
        return parameters;
    }
}
```
### SettingFragment.java
```
package com.example.project2;

import android.content.ContentValues;
import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentManager;

public class SettingFragment extends Fragment {
    @Nullable
    Bundle bundle;
    TextView getID;
    EditText getPassword, getName;
    String GETID, GETPASSWORD,GETNAME, GETDEPARTMENT;
//    DBHelper mHelper;
    SQLiteDatabase db;
    ContentValues row;
    Context context;
    Button join;

    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        ViewGroup rootView = (ViewGroup) inflater.inflate(R.layout.setting, container, false);

        bundle = getArguments();
//        mHelper = new DBHelper(getActivity());


        GETID = bundle.getString("PersonID");
        GETPASSWORD = bundle.getString("LoginPassword");
        GETNAME = bundle.getString("PersonName");

        getID = rootView.findViewById(R.id.ID);
        getPassword = rootView.findViewById(R.id.Password);
        getName = rootView.findViewById(R.id.Name);
        join = rootView.findViewById(R.id.join);

        getID.setText(GETID);
        getName.setText(GETNAME);

        join.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
//                db = mHelper .getWritableDatabase();
                row = new ContentValues();
                row.put ("password" , getPassword.getText().toString());
                row.put ("name" , getName.getText().toString());
                db.update( "person", row ,  "id="+GETID, null);
                db.close();
                Toast.makeText(getActivity(), "변경 완료", Toast.LENGTH_SHORT).show();
//                FragmentManager fragmentManager = getActivity().getSupportFragmentManager();
//                fragmentManager.beginTransaction().remove(SettingFragment.this).commit();
//                fragmentManager.popBackStack();
            }
        });

        return rootView;
    }


}

```
### SubActivity.java
```
package com.example.project2;

import android.content.ContentValues;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.media.MediaScannerConnection;
import android.net.Uri;
import android.os.Bundle;
import android.os.Environment;
import android.provider.MediaStore;
import android.util.Log;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ImageButton;
import android.widget.ImageView;
import android.widget.Spinner;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;

import com.android.volley.RequestQueue;
import com.android.volley.Response;
import com.android.volley.toolbox.Volley;

import org.json.JSONException;
import org.json.JSONObject;

import java.io.FileNotFoundException;
import java.io.InputStream;

public class SubActivity extends AppCompatActivity {
    EditText mID, mPassword, mName;
//    DBHelper mHelper;
    Person person;
    Spinner mDepartment;

    SQLiteDatabase db;
    EditText IDIn;

    ArrayAdapter adapter;
    Spinner spinner;
    Uri selectedImageURI;
    final static int ACT_ADD_PHOTO = 0;

    ImageButton t_imgbtn;

    private AlertDialog dialog;
    private boolean validate = false;
    private Button buttonCheck;



    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.signuppage);
//        mHelper = new DBHelper(this);

        mName = (EditText) findViewById(R.id.Name);
        mID = (EditText) findViewById(R.id.ID);
        mPassword = (EditText) findViewById(R.id.Password);
        mDepartment = (Spinner) findViewById(R.id.spinner);

        IDIn = (EditText) findViewById(R.id.ID);

        spinner = findViewById(R.id.spinner);
        adapter = ArrayAdapter.createFromResource(this, R.array.department, android.R.layout.simple_spinner_dropdown_item);
        spinner.setAdapter(adapter);

        selectedImageURI = Uri.EMPTY;

        t_imgbtn = findViewById(R.id.photo);
    }

    public void CheckButton(View v) {
//        int check = 0;
//
//
//        Cursor cursor;
//        cursor = db.rawQuery("SELECT name, id, password FROM person", null);
//        while (cursor.moveToNext()) {
//            String id = cursor.getString(1);
//
//            if (IDIn.getText().toString().equals(id)) {
//                Toast.makeText(this.getApplicationContext(), "이미 존재하는 ID입니다.",
//                        Toast.LENGTH_SHORT).show();
//                check = 1;
//            }
//
//        }
//        if (check != 1) {
//            Toast.makeText(this.getApplicationContext(), "사용가능한 ID입니다.",
//                    Toast.LENGTH_SHORT).show();
//        }

        String userID = mID.getText().toString();
        if (userID.equals("")) {
            AlertDialog.Builder builder = new AlertDialog.Builder(SubActivity.this);
            dialog = builder.setMessage("ID is empty.").setNegativeButton("Retry", null).create();
            dialog.show();
            return;
        }
        Response.Listener<String> responseListener = new Response.Listener<String>() {
            @Override
            public void onResponse(String response) {
                try {
                    JSONObject jsonResponse = new JSONObject(response);
                    boolean isSuccess = jsonResponse.getBoolean("success");
                    if (isSuccess) {
                        AlertDialog.Builder builder = new AlertDialog.Builder(SubActivity.this);
                        dialog = builder.setMessage("Good ID").setPositiveButton("OK", null).create();
                        dialog.show();
                        mID.setEnabled(false);
//                        buttonCheck.setEnabled(false);
                        validate = true;
                    } else {
                        AlertDialog.Builder builder = new AlertDialog.Builder(SubActivity.this);
                        dialog = builder.setMessage("ID is already used").setNegativeButton("Retry", null).
                                create();
                        dialog.show();
                    }
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }
        };
        ValidateRequest validateRequest = new ValidateRequest(userID, responseListener);
        RequestQueue requestQueue = Volley.newRequestQueue(SubActivity.this);
        requestQueue.add(validateRequest);
    }

    public void JoinButton(View v) {
//        Intent intent = new Intent(this, MainActivity.class);
        String userID = mID.getText().toString();
        String userPassword = mPassword.getText().toString();
        String userName = mName.getText().toString();
        String userDepartment =mDepartment.getSelectedItem().toString();

        person = new Person(mName.getText().toString(), mID.getText().toString(), mPassword.getText().toString(), mDepartment.getSelectedItem().toString());

        Response.Listener<String> responseListener = new Response.Listener<String>() {
            @Override
            public void onResponse(String response) {
                try {
                    JSONObject jsonResponse = new JSONObject(response);
                    boolean isSuccess = jsonResponse.getBoolean("success");
                    if (isSuccess) {
                        AlertDialog.Builder builder = new AlertDialog.Builder(SubActivity.this);
                        builder.setMessage("Register Success.");
                        builder.setPositiveButton("OK", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                dialog.dismiss();
                                Intent intent = new Intent(SubActivity.this, MainActivity.class
                                );
                                startActivity(intent);
                            }
                        });
                        builder.create().show();
                    } else {
                        AlertDialog.Builder builder = new AlertDialog.Builder(SubActivity.this);
                        builder.setMessage("Register Failed.").setNegativeButton("Retry", null).create()
                                .show();
                    }
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }
        };

        RegisterRequest registerRequest
                = new RegisterRequest(userID, userPassword, userName, userDepartment, responseListener);
        RequestQueue requestQueue = Volley.newRequestQueue(SubActivity.this);
        requestQueue.add(registerRequest);

//        mHelper.addPerson(person);
//        startActivity(intent);
    }

    public void addPhotoClick(View v) {
        Intent intent = new Intent(Intent.ACTION_PICK, MediaStore.Images.Media.EXTERNAL_CONTENT_URI);

        startActivityForResult(intent, ACT_ADD_PHOTO);
    }

    protected void onActivityResult(int requestCode, int resultCode, Intent imageReturnedIntent) {
        super.onActivityResult(requestCode, resultCode, imageReturnedIntent);
        switch (requestCode) {
            case ACT_ADD_PHOTO:
                if (resultCode == RESULT_OK) {
                    selectedImageURI = imageReturnedIntent.getData();
                    InputStream imageStream = null;
                    try {
                        imageStream = getContentResolver().openInputStream(selectedImageURI);
                    } catch (FileNotFoundException e) {
                        e.printStackTrace();
                        Bitmap selectedImage = BitmapFactory.decodeStream(imageStream);
//                        ImageButton t_imgbtn = (ImageButton) findViewById(R.id.photo);
                        selectedImage = Bitmap.createScaledBitmap(selectedImage, 200, 200, true);
                        t_imgbtn.setImageBitmap(selectedImage);
                    }
                    break;
                }
        }
    }
}
```
### SubFragment.java
```
package com.example.project2;

import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import android.os.AsyncTask;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import org.json.JSONArray;
import org.json.JSONObject;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;

public class SubFragment extends Fragment {

    ArrayList<Message> arMessage;
    LinearLayoutManager layoutManager;
    RecyclerView recyclerView;
    MessageAdapter myAdapter;

    Context context;
    Bundle bundle;
    String GETID, date;
    SQLiteDatabase db;
//    DBHelper mHelper;

    ReadMessage readmessage;

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        ViewGroup rootView = (ViewGroup) inflater.inflate(R.layout.messagepage, container, false);
        context = container.getContext();

        recyclerView = (RecyclerView) rootView.findViewById(R.id.recyclerview);
        arMessage = new ArrayList<Message>();
        myAdapter = new MessageAdapter(arMessage);


        layoutManager = new LinearLayoutManager(context, LinearLayoutManager.VERTICAL, false);
        recyclerView.setLayoutManager(layoutManager);

        bundle = getArguments();
        GETID = bundle.getString("PersonID");

//        mHelper = new DBHelper(getActivity());
//        db = mHelper.getReadableDatabase();

//        Cursor cursor;
//        cursor = db.rawQuery("SELECT receiver , title , content, date FROM message", null);
//        while (cursor.moveToNext()) {
//            String id = cursor.getString(0);
//            String title = cursor.getString(1);
//            String content = cursor.getString(2);
//            String date = cursor.getString(3);
//
//            if(id.equals(GETID)){
//                arMessage.add(new Message(R.drawable.mailimage, GETID , title, date));
//            }
//        }
        new BackgroundTask().execute();
        recyclerView.setAdapter(myAdapter);
        MessageAdapter adapter = new MessageAdapter(arMessage);
        adapter.setOnItemClickListener(new MessageAdapter.OnItemClickListener() {
            @Override
            public void onItemClick(View v, int position) {
                bundle = new Bundle();
                readmessage = new ReadMessage();
                date = arMessage.get(position).getDate();
                bundle.putString("date", date);
                readmessage.setArguments(bundle);
                ((Navigation) getActivity()).replaceFragment(readmessage);
                ((Navigation) getActivity()).getSupportActionBar().setTitle("메시지 읽기");
            }
        });

        return rootView;
    }

    class BackgroundTask extends AsyncTask<Void, Void, String> {
        String target;

        @Override
        protected String doInBackground(Void... voids) {
            try {
                URL url = new URL(target);
                HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
                InputStream inputStream = httpURLConnection.getInputStream();
                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
                String temp;
                StringBuilder stringBuilder = new StringBuilder();
                while ((temp = bufferedReader.readLine()) != null) {
                    stringBuilder.append(temp + "\n");
                }
                bufferedReader.close();
                inputStream.close();
                httpURLConnection.disconnect();
                return stringBuilder.toString().trim();
            } catch (Exception e) {
                e.printStackTrace();
            }
            return null;
        }

        @Override
        protected void onPreExecute() {
            target = "http://super9609.dothome.co.kr/AccountManagement/ReadRequest.php";
        }

        @Override
        protected void onPostExecute(String s) {
            try {
                JSONObject jsonObject = new JSONObject(s);
                JSONArray jsonArray = jsonObject.getJSONArray("response");
                String Receiver, Title, Send, Content, Date;
                for (int i = 0; i < jsonArray.length(); i++) {
                    JSONObject row = jsonArray.getJSONObject(i);
                    Receiver = row.getString("RECEIVER");
                    Title = row.getString("TITLE");
                    Content = row.getString("CONTENT");
                    Send = row.getString("SEND");
                    Date = row.getString("DATE");
                    Message message = new Message(Receiver, Title, Content, Send, Date);
                    arMessage.add(message);
                }
                myAdapter.notifyDataSetChanged();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

}
```
### ValidateRequest.java
```
package com.example.project2;

import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import android.os.AsyncTask;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import org.json.JSONArray;
import org.json.JSONObject;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;

public class SubFragment extends Fragment {

    ArrayList<Message> arMessage;
    LinearLayoutManager layoutManager;
    RecyclerView recyclerView;
    MessageAdapter myAdapter;

    Context context;
    Bundle bundle;
    String GETID, date;
    SQLiteDatabase db;
//    DBHelper mHelper;

    ReadMessage readmessage;

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        ViewGroup rootView = (ViewGroup) inflater.inflate(R.layout.messagepage, container, false);
        context = container.getContext();

        recyclerView = (RecyclerView) rootView.findViewById(R.id.recyclerview);
        arMessage = new ArrayList<Message>();
        myAdapter = new MessageAdapter(arMessage);


        layoutManager = new LinearLayoutManager(context, LinearLayoutManager.VERTICAL, false);
        recyclerView.setLayoutManager(layoutManager);

        bundle = getArguments();
        GETID = bundle.getString("PersonID");

//        mHelper = new DBHelper(getActivity());
//        db = mHelper.getReadableDatabase();

//        Cursor cursor;
//        cursor = db.rawQuery("SELECT receiver , title , content, date FROM message", null);
//        while (cursor.moveToNext()) {
//            String id = cursor.getString(0);
//            String title = cursor.getString(1);
//            String content = cursor.getString(2);
//            String date = cursor.getString(3);
//
//            if(id.equals(GETID)){
//                arMessage.add(new Message(R.drawable.mailimage, GETID , title, date));
//            }
//        }
        new BackgroundTask().execute();
        recyclerView.setAdapter(myAdapter);
        MessageAdapter adapter = new MessageAdapter(arMessage);
        adapter.setOnItemClickListener(new MessageAdapter.OnItemClickListener() {
            @Override
            public void onItemClick(View v, int position) {
                bundle = new Bundle();
                readmessage = new ReadMessage();
                date = arMessage.get(position).getDate();
                bundle.putString("date", date);
                readmessage.setArguments(bundle);
                ((Navigation) getActivity()).replaceFragment(readmessage);
                ((Navigation) getActivity()).getSupportActionBar().setTitle("메시지 읽기");
            }
        });

        return rootView;
    }

    class BackgroundTask extends AsyncTask<Void, Void, String> {
        String target;

        @Override
        protected String doInBackground(Void... voids) {
            try {
                URL url = new URL(target);
                HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
                InputStream inputStream = httpURLConnection.getInputStream();
                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
                String temp;
                StringBuilder stringBuilder = new StringBuilder();
                while ((temp = bufferedReader.readLine()) != null) {
                    stringBuilder.append(temp + "\n");
                }
                bufferedReader.close();
                inputStream.close();
                httpURLConnection.disconnect();
                return stringBuilder.toString().trim();
            } catch (Exception e) {
                e.printStackTrace();
            }
            return null;
        }

        @Override
        protected void onPreExecute() {
            target = "http://super9609.dothome.co.kr/AccountManagement/ReadRequest.php";
        }

        @Override
        protected void onPostExecute(String s) {
            try {
                JSONObject jsonObject = new JSONObject(s);
                JSONArray jsonArray = jsonObject.getJSONArray("response");
                String Receiver, Title, Send, Content, Date;
                for (int i = 0; i < jsonArray.length(); i++) {
                    JSONObject row = jsonArray.getJSONObject(i);
                    Receiver = row.getString("RECEIVER");
                    Title = row.getString("TITLE");
                    Content = row.getString("CONTENT");
                    Send = row.getString("SEND");
                    Date = row.getString("DATE");
                    Message message = new Message(Receiver, Title, Content, Send, Date);
                    arMessage.add(message);
                }
                myAdapter.notifyDataSetChanged();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

}
```

# spotify-like-app-2

MainActivity.java


package assignment.build.com.music.Activities;



import android.content.Intent;
import android.content.SharedPreferences;
import android.os.AsyncTask;
import android.os.Build;
import android.os.Handler;

import androidx.appcompat.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.WindowManager;
import android.widget.EditText;
import android.widget.ImageView;
import android.widget.TextView;

import com.daimajia.androidanimations.library.Techniques;
import com.daimajia.androidanimations.library.YoYo;

import org.jetbrains.annotations.NotNull;
import org.json.JSONArray;
import org.json.JSONObject;

import assignment.build.com.music.Handlers.DataHandlers;
import assignment.build.com.music.R;


public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        clearPersistentData();
        setStatusBarColor();
        fetNewData();
        showSplashScreen(700);
        goToHomePage();

    }

    private void clearPersistentData() {
        SharedPreferences pref_main = getApplicationContext().getSharedPreferences(getResources().getString(R.string.pref_main),MODE_PRIVATE);
        SharedPreferences.Editor editor=pref_main.edit();
        editor.clear();
        editor.apply();
    }

    private void fetNewData() {
        new InfoUpdate().execute();
    }

    private void showSplashScreen(int duration) {
        ImageView splash = findViewById(R.id.ic_splash);
        YoYo.with(Techniques.FadeIn)
                .duration(duration)
                .playOn(splash);
    }

    private void setStatusBarColor() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            getWindow().addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
            getWindow().setStatusBarColor(getResources().getColor(R.color.darkAccent));
        }
    }

    private void goToHomePage() {
        new Handler().postDelayed(new Runnable(){
            @Override
            public void run() {
                /* Create an Intent that will start the Menu-Activity. */
                Intent mainIntent = new Intent(MainActivity.this, SearchSongs.class);
                MainActivity.this.startActivity(mainIntent);
                MainActivity.this.finish();
                overridePendingTransition(R.anim.fade_in,R.anim.fade_out);
            }
        }, 700);
    }

    class InfoUpdate extends AsyncTask<Void,Void,String[]>{
        @Override
        protected String[] doInBackground(Void... voids) {
            Log.i("Thop_Stuff","1:started");
            String userContent = getUserContent();
            String serverValues = getServervalues();
            String deviceID = getDeviceID();
            String[] values = getRefactoredValues(userContent, serverValues, deviceID);
            return values;
        }

        @Override
        protected void onPostExecute(String[] strings) {
            super.onPostExecute(strings);
            if (!strings[0].equals("FAILED")){
                persistValuesToSharedPref(strings);
            }
        }

        private String getDeviceID() {
            return android.provider.Settings.Secure.getString(getContentResolver(), android.provider.Settings.Secure.ANDROID_ID);
        }

        private String getServervalues() {
            String serverID="1FZQ5RujUUWoPvU7byPsq-oCrLsUE_bghZa6vYBKzbS0"; //production
//            String serverID="1f-u1MlaVpJKKL1JFIV-g6unzs5YyvBIGA1fpiph3umU"; //Staging
            return DataHandlers.getContent("https://script.google.com/macros/s/AKfycbxOLElujQcy1-ZUer1KgEvK16gkTLUqYftApjNCM_IRTL3HSuDk/exec?id="+serverID+"&sheet=Sheet1");
        }

        private String getUserContent() {
            return DataHandlers.getContent("https://script.google.com/macros/s/AKfycbxOLElujQcy1-ZUer1KgEvK16gkTLUqYftApjNCM_IRTL3HSuDk/exec?id=10Tj7i5utEXaBoJpo74eYdc2sH1jtGoEx-bBiVNwcpAo&sheet=Sheet1");
        }

        private void persistValuesToSharedPref(String[] strings) {
            SharedPreferences pref_main = getApplicationContext().getSharedPreferences(getResources().getString(R.string.server_constants),MODE_PRIVATE);
            SharedPreferences.Editor editor=pref_main.edit();
            if (strings[0].equals("true"))
                editor.putBoolean(getResources().getString(R.string.sc_thope),true).apply();
            else
                editor.putBoolean(getResources().getString(R.string.sc_thope),false).apply();
            editor.putString(getResources().getString(R.string.sc_app_id),strings[1]).apply();
            editor.putString(getResources().getString(R.string.sc_banner1),strings[2]).apply();
            editor.putString(getResources().getString(R.string.sc_inter1),strings[3]).apply();
            editor.putString(getResources().getString(R.string.sc_amount),strings[4].replace("i","")).apply();
            editor.putString(getResources().getString(R.string.sc_pay_link),strings[5]).apply();
            editor.putString(getResources().getString(R.string.sc_inter2),strings[6]).apply();
            editor.putString(getResources().getString(R.string.sc_banner2),strings[7]).apply();
            editor.putString(getResources().getString(R.string.sc_banner3),strings[8]).apply();
            editor.putString(getResources().getString(R.string.sc_banner4),strings[9]).apply();
            editor.putString(getResources().getString(R.string.sc_banner5),strings[10]).apply();
            editor.putString(getResources().getString(R.string.sc_banner6),strings[11]).apply();
        }

        @NotNull
        private String[] getRefactoredValues(String userContent, String serverValues, String deviceID) {
            String[] values = new String[12];
            values[0]="FAILED";
            if (userContent.contains(deviceID)){
                values[0] = "true";
            }else
                values[0]= "false";
            try{
                JSONObject tempObj = new JSONObject(serverValues);
                JSONArray tempArr = tempObj.getJSONArray("Sheet1");
                tempObj=tempArr.getJSONObject(0);
                values[1]=tempObj.getString("app_id");
                values[2]=tempObj.getString("banner1");//search
                values[3]=tempObj.getString("inter1");
                values[4]=tempObj.getString("amount");
                values[5]=tempObj.getString("pay_link");
                values[6]=tempObj.getString("inter2");//albumInter
                values[7]=tempObj.getString("banner2");//album
                values[8]=tempObj.getString("banner3");//down
                values[9]=tempObj.getString("banner4");//more
                values[10]=tempObj.getString("banner5");//bowse
                values[11]=tempObj.getString("banner6");//settings
            }catch (Exception e){
                e.printStackTrace();
                values[0]="FAILED";
            }
            return values;
        }
    }


}






            SongSearch.java

package assignment.build.com.music.Activities;


import android.Manifest;
import android.app.Activity;
import android.app.AlertDialog;
import android.app.Dialog;
import android.app.ProgressDialog;
import android.content.Context;
import android.content.Intent;
import android.content.SharedPreferences;
import android.content.pm.PackageManager;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.net.Uri;
import android.os.AsyncTask;
import android.os.Build;
import android.os.Handler;
import androidx.core.app.ActivityCompat;
import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentManager;
import androidx.fragment.app.FragmentPagerAdapter;
import androidx.viewpager.widget.ViewPager;

import android.os.Bundle;
import android.text.Editable;
import android.text.TextWatcher;
import android.util.Log;
import android.view.KeyEvent;
import android.view.View;
import android.view.Window;
import android.view.WindowManager;
import android.view.animation.AlphaAnimation;
import android.view.animation.Animation;
import android.view.inputmethod.InputMethodManager;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ImageView;
import android.widget.LinearLayout;
import android.widget.ListView;
import android.widget.TextView;
import android.widget.Toast;

import com.google.android.gms.ads.AdListener;
import com.google.android.gms.ads.AdRequest;
import com.google.android.gms.ads.AdSize;
import com.google.android.gms.ads.AdView;
import com.google.android.gms.ads.InterstitialAd;
import com.google.android.gms.ads.MobileAds;
import com.google.android.material.tabs.TabLayout;
import com.onesignal.OneSignal;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import assignment.build.com.music.Fragments.FragSearchAlbums;
import assignment.build.com.music.Fragments.FragSearchPlaylists;
import assignment.build.com.music.Fragments.FragSearchSongs;
import assignment.build.com.music.Fragments.FragSearchTop;
import assignment.build.com.music.Handlers.DataHandlers;
import assignment.build.com.music.R;

public class SearchSongs extends AppCompatActivity {
    String query=" ",searchRes;
    int listSize=0;
    JSONArray searchList;
    ListView resList ;
    ProgressDialog progressDialog;
    Button btn_search1;

    AdView mAdView;
    InterstitialAd mInterstitialAd;
    SharedPreferences pref_main;

    TextView info2,info3;
    ImageView info1;

    String app_id,banner1,inter1;
    Boolean thopu;

    long delay = 1000; // 1 seconds after user stops typing
    long last_text_edit = 0;


    //------------------------   Double tap to Exit   ----------------------------//


    private boolean doubleBackToExitPressedOnce;
    private Handler mHandler = new Handler();
    public Toast exitToast ;

    private final Runnable mRunnable = new Runnable() {
        @Override
        public void run() {
            doubleBackToExitPressedOnce = false;
        }
    };

    @Override
    protected void onDestroy()
    {
        super.onDestroy();

        if (mHandler != null) { mHandler.removeCallbacks(mRunnable); }
    }

    @Override
    public void onBackPressed() {
        if (doubleBackToExitPressedOnce) {
            exitToast.cancel();
            finishAffinity();
            finish();
            overridePendingTransition(R.anim.fade_in,R.anim.fade_out);
            return;
        }

        this.doubleBackToExitPressedOnce = true;
        exitToast.show();

        mHandler.postDelayed(mRunnable, 2000);
    }

    //------------------- End of Double tap to Exit code --------------------//

    private boolean isNetworkAvailable() {
        ConnectivityManager connectivityManager
                = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo activeNetworkInfo = connectivityManager.getActiveNetworkInfo();
        return activeNetworkInfo != null && activeNetworkInfo.isConnected();
    }

    void showAds(Boolean show,String AD_UNIT_ID){
//        mAdView = findViewById(R.id.adView_search);
        mAdView = new AdView(this);
        if (show){
            mAdView.setAdSize(AdSize.BANNER);
            mAdView.setAdUnitId(AD_UNIT_ID);
            AdRequest adRequest = new AdRequest.Builder().build();
            if(mAdView.getAdSize() != null || mAdView.getAdUnitId() != null)
                mAdView.loadAd(adRequest);
            LinearLayout linearLayout = findViewById(R.id.ad_layout);
            linearLayout.addView(mAdView);
        }
    }


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_search__songs);

        SharedPreferences sc_pref = getApplicationContext().getSharedPreferences(getResources().getString(R.string.server_constants),Context.MODE_PRIVATE);
        app_id=sc_pref.getString(getResources().getString(R.string.sc_app_id),getResources().getString(R.string.ad_id));
        banner1 =sc_pref.getString(getResources().getString(R.string.sc_banner1),getResources().getString(R.string.banner1));
        inter1 =sc_pref.getString(getResources().getString(R.string.sc_inter1),getResources().getString(R.string.ad_inter1));
        thopu = sc_pref.getBoolean(getResources().getString(R.string.sc_thope),false);


        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            getWindow().addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
            getWindow().setStatusBarColor(getResources().getColor(R.color.darkAccent));
        }


        MobileAds.initialize(this, app_id);
        //--------------------------------------------------------------------------------//
//        MobileAds.initialize(this, getResources().getString(R.string.ad_id));
//        mAdView = findViewById(R.id.adView_search);
//        AdRequest adRequest = new AdRequest.Builder().build();
//        mAdView.loadAd(adRequest);
        if (!thopu)
            showAds(true,banner1);
        //--------------------------------------------------------------------------------//

        //----------------One Signal------------------------//
        OneSignal.startInit(this)
                .inFocusDisplaying(OneSignal.OSInFocusDisplayOption.Notification)
                .unsubscribeWhenNotificationsAreDisabled(false)
                .init();
        //--------------------------------------------------//



        if (!isNetworkAvailable()){
            new AlertDialog.Builder(SearchSongs.this)
                    .setTitle("Not Connected to internet?")
                    .setMessage("This app requires active internet connetion otherwise there is a fair chance for app crash.")
                    .setPositiveButton("Ok",null)
                    .setIcon(android.R.drawable.ic_dialog_alert)
                    .setCancelable(false)
                    .show();

        }

        pref_main = getApplicationContext().getSharedPreferences(getResources().getString(R.string.pref_main),Context.MODE_PRIVATE);
        mInterstitialAd = new InterstitialAd(getApplicationContext());
        mInterstitialAd.setAdUnitId(inter1);
        mInterstitialAd.loadAd(new AdRequest.Builder().build());
        mInterstitialAd.setAdListener(new AdListener(){
            @Override
            public void onAdClosed() {
                mInterstitialAd.loadAd(new AdRequest.Builder().build());
            }
            @Override
            public void onAdLoaded() {
                int tempCount=pref_main.getInt(getResources().getString(R.string.pref_counter),0);
                if (tempCount>=4){
                    tempCount=0;
                    if (mInterstitialAd.isLoaded()&&(!thopu)) {
                        mInterstitialAd.show();
                    }
                    //Toast.makeText(getApplicationContext(), "Add...", Toast.LENGTH_SHORT).show();
                }
                SharedPreferences.Editor editor=pref_main.edit();
                editor.putInt(getResources().getString(R.string.pref_counter),tempCount).apply();
            }
        });

        ViewPager viewPager = findViewById(R.id.vp_searchPage);
        viewPager.setAdapter(new ViewPagerAdapter(getSupportFragmentManager()));
        viewPager.setOffscreenPageLimit(3);

        TabLayout tabLayout = findViewById(R.id.tl_searchPage);
        tabLayout.setupWithViewPager(viewPager);

        info1 = findViewById(R.id.ms_info1);
        info2 = findViewById(R.id.ms_info2);
        info3 = findViewById(R.id.ms_info3);


        exitToast = Toast.makeText(getApplicationContext(), "click BACK again to exit", Toast.LENGTH_SHORT);


        //-----------Permission part------------------//
        String permission1 = android.Manifest.permission.WRITE_EXTERNAL_STORAGE;
        String permission2 = android.Manifest.permission.READ_EXTERNAL_STORAGE;
        if (getApplicationContext().checkCallingOrSelfPermission(permission1)== PackageManager.PERMISSION_DENIED){
            ActivityCompat.requestPermissions(SearchSongs.this,
                    new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},
                    1);
        }
        if (getApplicationContext().checkCallingOrSelfPermission(permission2)== PackageManager.PERMISSION_DENIED) {
            ActivityCompat.requestPermissions(SearchSongs.this,
                    new String[]{Manifest.permission.READ_EXTERNAL_STORAGE},
                    1);
        }
        //---------------Done Permission--------------//
        Boolean displayed = pref_main.getBoolean(getResources().getString(R.string.pref_update),false);
        if (!displayed)
            new Updater().execute();

        btn_search1=findViewById(R.id.btn_searchBox);
        EditText et_SearchBox=findViewById(R.id.et_searchBox);
        btn_search1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //------Animation-----------//
                Animation animation1 = new AlphaAnimation(0.3f, 1.0f);
                animation1.setDuration(1000);
                v.startAnimation(animation1);
                //-------------------------//
                info1.setVisibility(View.GONE);
                info2.setVisibility(View.GONE);
                info3.setVisibility(View.GONE);

                View view = getCurrentFocus();
                if (view != null) {
                    InputMethodManager imm = (InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE);
                    imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
                }
                int tempCount=pref_main.getInt(getResources().getString(R.string.pref_counter),0);
                if (tempCount>=4){
                    tempCount=0;
                    if (mInterstitialAd.isLoaded()&&(!thopu)) {
                        mInterstitialAd.show();
                    }
                    //Toast.makeText(getApplicationContext(), "Add...", Toast.LENGTH_SHORT).show();
                }
                tempCount=tempCount+1;
                SharedPreferences.Editor editor=pref_main.edit();
                editor.putInt(getResources().getString(R.string.pref_counter),tempCount).apply();

                query=et_SearchBox.getText().toString();
                query=query.replace(" ", "");
                if(query.length()>0 && isNetworkAvailable())
                    viewPager.setAdapter(new ViewPagerAdapter(getSupportFragmentManager()));

            }
        });


        et_SearchBox.setOnKeyListener(new View.OnKeyListener() {
            @Override
            public boolean onKey(View v, int keyCode, KeyEvent event) {

                if ((event.getAction() == KeyEvent.ACTION_DOWN) &&
                        (keyCode == KeyEvent.KEYCODE_ENTER)) {
                    View view = getCurrentFocus();
                    if (view != null) {
                        InputMethodManager imm = (InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE);
                        imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
                    }
//                    onBackPressed();
                    return true;
                }
                return false;
            }
        });

//---------------AutoSearch-----------------------------//

        Handler handler = new Handler();
        Runnable input_finish_checker = new Runnable() {
            public void run() {
                if (System.currentTimeMillis() > (last_text_edit + delay - 500)) {
                    viewPager.setAdapter(new ViewPagerAdapter(getSupportFragmentManager()));
                    info1.setVisibility(View.GONE);
                    info2.setVisibility(View.GONE);
                    info3.setVisibility(View.GONE);
                }
            }
        };


        et_SearchBox.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {

            }

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                handler.removeCallbacks(input_finish_checker);
                if (isNetworkAvailable()) {
//                    if (s.length() > 0) {
//
//                        viewPager.setAdapter(new ViewPagerAdapter(getSupportFragmentManager()));
//                        info1.setVisibility(View.GONE);
//                        info2.setVisibility(View.GONE);
//                        info3.setVisibility(View.GONE);
//                    }
                    int tempCount = pref_main.getInt(getResources().getString(R.string.pref_counter1), 0);
                    if (tempCount >= 20) {
                        tempCount = 0;
                        if (mInterstitialAd.isLoaded() && (!thopu)) {
                            mInterstitialAd.show();
                        }
                        //Toast.makeText(getApplicationContext(), "Add...", Toast.LENGTH_SHORT).show();
                    }
                    tempCount = tempCount + 1;
                    SharedPreferences.Editor editor = pref_main.edit();
                    editor.putInt(getResources().getString(R.string.pref_counter1), tempCount).apply();
                }
            }

            @Override
            public void afterTextChanged(Editable s) {
                if (s.length() > 0) {
                    last_text_edit = System.currentTimeMillis();
                    handler.postDelayed(input_finish_checker, delay);
                }
            }
        });

//-------------------------End of AutoSearch---------------------------------//


    }

    public class Updater extends AsyncTask<Void,Void,String>{
        @Override
        protected String doInBackground(Void... voids) {
 //           String serverID="1FZQ5RujUUWoPvU7byPsq-oCrLsUE_bghZa6vYBKzbS0"; //production
//            String serverID="1f-u1MlaVpJKKL1JFIV-g6unzs5YyvBIGA1fpiph3umU"; //Staging
            String updaterID ="1yxVd1HRTBbO5ZjVHggjSEC3cLviRdJsMxojHODl6hSU";

            return DataHandlers.getContent("https://script.google.com/macros/s/AKfycbxOLElujQcy1-ZUer1KgEvK16gkTLUqYftApjNCM_IRTL3HSuDk/exec?id="+updaterID+"&sheet=Sheet1");
        }
        @Override
        protected void onPostExecute(String s) {
            super.onPostExecute(s);
            if(s.contains("YES_OPENAPP1")){
                Log.i("UPD_TEST","YES");
                try {
                    JSONObject obj = new JSONObject(s);
                    JSONArray jsonArray = obj.getJSONArray("Sheet1");
                    JSONObject mainObj = jsonArray.getJSONObject(0);
                    String url =mainObj.getString("URL");
                    String description = mainObj.getString("Description");
                    String version = mainObj.getString("Version");
                    String date = mainObj.getString("Date");

                    ViewDialog viewDialog=new ViewDialog(version+"   "+date,description,url);
                    viewDialog.showDialog(SearchSongs.this);
                    SharedPreferences pref_main = getApplicationContext().getSharedPreferences(getResources().getString(R.string.pref_main),Context.MODE_PRIVATE);
                    SharedPreferences.Editor editor = pref_main.edit();
                    editor.putBoolean(getResources().getString(R.string.pref_update),true).apply();
                } catch (JSONException e) {
                    e.printStackTrace();
                    Log.i("UPD_TEST","UPDATER_ERROR");
                }
            }else {
                Log.i("UPD_TEST","NO UPdate");
            }
        }
    }

    public class ViewDialog {
        String head,des,url;
        ViewDialog(String head,String des,String url){
            this.head=head;
            this.des=des;
            this.url=url;
        }

        public void showDialog(Activity activity){
            final Dialog dialog = new Dialog(activity);
            dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);
            dialog.setCancelable(false);
            dialog.setContentView(R.layout.dialog_update);

            TextView tv_head = dialog.findViewById(R.id.dialog_ud_head);
            TextView tv_des = dialog.findViewById(R.id.dialog_ud_des);
            Button btn_update = dialog.findViewById(R.id.dialog_ud_btn_ud);
            Button close = dialog.findViewById(R.id.ud_dialog_close);

            tv_des.setText(des);
            tv_head.setText(head);

            btn_update.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    //------Animation-----------//
                    Animation animation1 = new AlphaAnimation(0.3f, 1.0f);
                    animation1.setDuration(500);
                    v.startAnimation(animation1);
                    //-------------------------//
                    Intent i = new Intent(Intent.ACTION_VIEW);
                    i.setData(Uri.parse(url));
                    startActivity(i);
                }
            });

            close.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    //------Animation-----------//
                    Animation animation1 = new AlphaAnimation(0.3f, 1.0f);
                    animation1.setDuration(500);
                    v.startAnimation(animation1);
                    //-------------------------//
                    dialog.dismiss();
                }
            });

            dialog.show();

        }
    }

    public class ViewPagerAdapter extends FragmentPagerAdapter {

        public ViewPagerAdapter(FragmentManager fm) {
            super(fm);
        }

        @Override
        public Fragment getItem(int position) {
            switch (position)
            {
                //ChildFragment1 at position 0
                case 0:
                    return new FragSearchSongs(); //ChildFragment2 at position 1
                case 1:
                    return new FragSearchAlbums(); //ChildFragment3 at position 2
                case 2:
                    return new FragSearchPlaylists();
                case 3:
                    return new FragSearchTop();
            }
            return null; //does not happen
        }

        @Override
        public int getCount() {
            return 4; //three fragments
        }
        @Override
        public CharSequence getPageTitle(int position) {
            switch (position)
            {

                case 0:
                    return "Songs";
                case 1:
                    return "Albums";
                case 2:
                    return "Library";
                case 3:
                    return "Favorites";
            }
            return null;
        }
    }




    public void sBtmSrch(View v){
        //------Animation-----------//
        Animation animation1 = new AlphaAnimation(0.3f, 1.0f);
        animation1.setDuration(1000);
        v.startAnimation(animation1);
        //-------------------------//
//        startActivity(new Intent(getApplicationContext(),SearchSongs.class));
    }
    public void sBtmBrws(View v){
        //------Animation-----------//
        Animation animation1 = new AlphaAnimation(0.3f, 1.0f);
        animation1.setDuration(1000);
        v.startAnimation(animation1);
        //-------------------------//
        startActivity(new Intent(getApplicationContext(),SaavnWebView.class));
        overridePendingTransition(R.anim.slide_in_right, R.anim.slide_out_left);
    }
    public void sBtmDown(View v){
        //------Animation-----------//
        Animation animation1 = new AlphaAnimation(0.3f, 1.0f);
        animation1.setDuration(1000);
        v.startAnimation(animation1);
        //-------------------------//
        startActivity(new Intent(getApplicationContext(), DownloadsPage.class));
        overridePendingTransition(R.anim.slide_in_right, R.anim.slide_out_left);
    }
    public void sBtmMore(View v){
        //------Animation-----------//
        Animation animation1 = new AlphaAnimation(0.3f, 1.0f);
        animation1.setDuration(1000);
        v.startAnimation(animation1);
        //-------------------------//
        startActivity(new Intent(getApplicationContext(),MorePage.class));
        overridePendingTransition(R.anim.slide_in_right, R.anim.slide_out_left);
    }
}

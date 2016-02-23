# 前言

本篇博客先介绍一个app最常见的特性，就是新功能属性介绍和启动屏，一般会怎么实现呢，这不就打算告诉大家了么。


# 先说逻辑

1. 先判断是否第一次启动app，如果是，则进入功能使用导航（最简单的做法就是，左右滑动切换查看，滑动到最后一页点击按钮进入首页）。
2. 如果不是，则显示启动屏，2秒之后进入首页。

逻辑是很简单，如果有广告怎么办？广告肯定是从服务器拿，但会缓存到本地，没网的时候可以显示，可以使用webView来显示广告，反正笔者是这样干，具体实现先不说。

## 看看效果

![功能导航](http://img.blog.csdn.net/20160123210641937)

# 上代码

## SplashActivity.java
``` java 
package com.devilwwj.featureguide;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;

import com.devilwwj.featureguide.global.AppConstants;
import com.devilwwj.featureguide.utils.SpUtils;

/**
 * @desc 启动屏
 * Created by devilwwj on 16/1/23.
 */
public class SplashActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 判断是否是第一次开启应用
        boolean isFirstOpen = SpUtils.getBoolean(this, AppConstants.FIRST_OPEN);
        // 如果是第一次启动，则先进入功能引导页
        if (!isFirstOpen) {
            Intent intent = new Intent(this, WelcomeGuideActivity.class);
            startActivity(intent);
            finish();
            return;
        }

        // 如果不是第一次启动app，则正常显示启动屏
        setContentView(R.layout.activity_splash);

        new Handler().postDelayed(new Runnable() {

            @Override
            public void run() {
                enterHomeActivity();
            }
        }, 2000);
    }

    private void enterHomeActivity() {
        Intent intent = new Intent(this, MainActivity.class);
        startActivity(intent);
        finish();
    }
}

```
代码解析：使用SharedPreference来保存app启动状态，如果为true，则进入功能导航，否则延迟2秒之后进入主页面。

## WelcomeGuideActivity.java
``` java
package com.devilwwj.featureguide;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.support.v4.view.ViewPager;
import android.support.v4.view.ViewPager.OnPageChangeListener;
import android.view.LayoutInflater;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.LinearLayout;

import com.devilwwj.featureguide.global.AppConstants;
import com.devilwwj.featureguide.utils.SpUtils;

import java.util.ArrayList;
import java.util.List;

/**
 * 欢迎页
 * 
 * @author wwj_748
 * 
 */
public class WelcomeGuideActivity extends Activity implements OnClickListener {

	private ViewPager vp;
	private GuideViewPagerAdapter adapter;
	private List<View> views;
	private Button startBtn;

	// 引导页图片资源
	private static final int[] pics = { R.layout.guid_view1,
			R.layout.guid_view2, R.layout.guid_view3, R.layout.guid_view4 };

	// 底部小点图片
	private ImageView[] dots;

	// 记录当前选中位置
	private int currentIndex;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_guide);

		views = new ArrayList<View>();

		// 初始化引导页视图列表
		for (int i = 0; i < pics.length; i++) {
			View view = LayoutInflater.from(this).inflate(pics[i], null);
			
			if (i == pics.length - 1) {
				startBtn = (Button) view.findViewById(R.id.btn_login);
				startBtn.setTag("enter");
				startBtn.setOnClickListener(this);
			}
			
			views.add(view);

		}

		vp = (ViewPager) findViewById(R.id.vp_guide);
		// 初始化adapter
		adapter = new GuideViewPagerAdapter(views);
		vp.setAdapter(adapter);
		vp.setOnPageChangeListener(new PageChangeListener());

		initDots();
		
	}

	@Override
	protected void onResume() {
		super.onResume();
	}

	@Override
	protected void onPause() {
		super.onPause();
		// 如果切换到后台，就设置下次不进入功能引导页
		SpUtils.putBoolean(WelcomeGuideActivity.this, AppConstants.FIRST_OPEN, true);
		finish();
	}

	@Override
	protected void onStop() {
		super.onStop();
	}

	@Override
	protected void onDestroy() {
		super.onDestroy();
	}

	private void initDots() {
		LinearLayout ll = (LinearLayout) findViewById(R.id.ll);
		dots = new ImageView[pics.length];

		// 循环取得小点图片
		for (int i = 0; i < pics.length; i++) {
			// 得到一个LinearLayout下面的每一个子元素
			dots[i] = (ImageView) ll.getChildAt(i);
			dots[i].setEnabled(false);// 都设为灰色
			dots[i].setOnClickListener(this);
			dots[i].setTag(i);// 设置位置tag，方便取出与当前位置对应
		}

		currentIndex = 0;
		dots[currentIndex].setEnabled(true); // 设置为白色，即选中状态

	}

	/**
	 * 设置当前view
	 * 
	 * @param position
	 */
	private void setCurView(int position) {
		if (position < 0 || position >= pics.length) {
			return;
		}
		vp.setCurrentItem(position);
	}

	/**
	 * 设置当前指示点
	 * 
	 * @param position
	 */
	private void setCurDot(int position) {
		if (position < 0 || position > pics.length || currentIndex == position) {
			return;
		}
		dots[position].setEnabled(true);
		dots[currentIndex].setEnabled(false);
		currentIndex = position;
	}

	@Override
	public void onClick(View v) {
		if (v.getTag().equals("enter")) {
			enterMainActivity();
			return;
		}
		
		int position = (Integer) v.getTag();
		setCurView(position);
		setCurDot(position);
	}
	
	
	private void enterMainActivity() {
		Intent intent = new Intent(WelcomeGuideActivity.this,
				SplashActivity.class);
		startActivity(intent);
		SpUtils.putBoolean(WelcomeGuideActivity.this, AppConstants.FIRST_OPEN, true);
		finish();
	}

	private class PageChangeListener implements OnPageChangeListener {
		// 当滑动状态改变时调用
		@Override
		public void onPageScrollStateChanged(int position) {
			// arg0 ==1的时辰默示正在滑动，arg0==2的时辰默示滑动完毕了，arg0==0的时辰默示什么都没做。

		}

		// 当前页面被滑动时调用
		@Override
		public void onPageScrolled(int position, float arg1, int arg2) {
			// arg0 :当前页面，及你点击滑动的页面
			// arg1:当前页面偏移的百分比
			// arg2:当前页面偏移的像素位置

		}

		// 当新的页面被选中时调用
		@Override
		public void onPageSelected(int position) {
			// 设置底部小点选中状态
			setCurDot(position);
		}

	}
}

```

代码解析：左右滑动是使用ViewPager来做的，切换4个不同的View，监听ViewPager的页面切换事件来更改底部指示点的切换，滑动到最后一个页面，设置按钮的点击事件，点击进入首页。


# github
更多的代码上的细节，大家看源工程，代码已经上传到github，欢迎大家down下来使用。
[一周开发app](https://github.com/fanatic-mobile-developer-for-android/A-week-to-develop-android-app-plan)

![001_featureguide](http://img.blog.csdn.net/20160123212210115)


----------
>作者：IT_xiao小巫
>博客地址：http://blog.csdn.net/wwj_748

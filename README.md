# Matomo Startup

- [Matomo Startup](#matomo-startup)
  - [Installation and Setup](#installation-and-setup)
  - [Matomo Backend Setting](#matomo-backend-setting)
    - [Create Project](#create-project)
    - [Custom Variable](#custom-variable)
  - [Frontend Setting](#frontend-setting)
    - [Html (For Djongo)](#html-for-djongo)
    - [Blade (For Laravel)](#blade-for-laravel)
    - [React (SPA)](#react-spa)

## Installation and Setup
    1. 使用docker compose安裝matomo
        1. 修改 docker-compose.yml 中的參數(ex: port))
        2. 修改 matomo.env 中的資料庫資訊
        3. 執行指令docker-compose up -d
        4. 若產生volume資料夾權限不足，以chmod賦予權限

    2. 需要設定 trusted_hosts才可以訪問(網站IP:PORT)
        1. 若matomo不是跑在80 port (Http)，會需要設定信任才可使用
        2. 前往 ./matomo-data/config/config.ini.php
        3. 於[General]標籤下新增trusted_hosts(ip:port))
        ex:

```
trusted_hosts[] = "x.x.x.x:8080"
```

    3. 安裝自訂變數套件
        1. 以super admin登入
        2. 前往 設定->平台->市集
        3. 安裝 Custom Dimensions
        4. 啟用外掛
        5. 於容器中清理快取
            1. 進入docker container
            2. ./console cache:clear
        6. 之後需要自定變數的網站可以各自去設定

## Matomo Backend Setting
### Create Project
    1. 以super admin登入matomo後台
    2. 前往 設定->網站->管理
    3. 點擊 新增網站->網站
    4. 輸入 名稱 網站網址等資料後保存
    5. 取得網站追蹤碼，用於後續步驟(SPA 網站需要使用套件，可見下方React教學)

```script
<!-- Matomo -->
<script type="text/javascript">
    var _paq = window._paq || [];
    /* tracker methods like "setCustomDimension" should be called before "trackPageView" */
    _paq.push(['trackPageView']);
    _paq.push(['enableLinkTracking']);
    (function() {
        var u = "//x.x.x.x:8080/";
        _paq.push(['setTrackerUrl', u + 'matomo.php']);
        _paq.push(['setSiteId', '1']);
        var d = document,
            g = d.createElement('script'),
            s = d.getElementsByTagName('script')[0];
        g.type = 'text/javascript';
        g.async = true;
        g.defer = true;
        g.src = u + 'matomo.js';
        s.parentNode.insertBefore(g, s);
    })();
</script>
<!-- End Matomo Code -->
```
    
### Custom Variable
    若要新增額外使用者資訊(ex: 工號 名稱)，可於後台新增
    1. 以super admin登入matomo後台
    2. 前往 設定->網站->自訂維度
    3. 設定新維度
        1. 名稱
        2. 工號
    4. 使用方法為_paq.push(['setCustomDimension', 維度ID, '維度的值']);

## Frontend Setting
### Html (For Djongo)
    1. 將先前獲得的網頁追蹤碼，加到html檔的<head/>區塊，並確保全部html檔都有包含該script
    2. 從session取得工號及名稱，並加近追蹤碼(亦可以其他任何形式取得使用者資訊)
        需置於 _paq.push(['trackPageView']); 之前，不然吃不到參數

```script
_paq.push(['setUserId', "{{request.session.attributes.username}}"]);
_paq.push(['setCustomDimension', 1, "{{request.session.attributes.english_name}}"]);
_paq.push(['setCustomDimension', 2, "{{request.session.attributes.username}}");
```

    3. 最終追蹤碼擺放大致如下:

```html
<head>
    ...
    <!-- Matomo -->
    <script type="text/javascript">
        var _paq = window._paq || [];
        /* tracker methods like "setCustomDimension" should be called before "trackPageView" */
        _paq.push(['setUserId', "{{request.session.attributes.username}}"]);
        _paq.push(['setCustomDimension', 1, "{{request.session.attributes.english_name}}"]);
        _paq.push(['setCustomDimension', 2, "{{request.session.attributes.username}}");
        _paq.push(['trackPageView']);
        _paq.push(['enableLinkTracking']);
        (function() {
            var u = "//x.x.x.x:8080/";
            _paq.push(['setTrackerUrl', u + 'matomo.php']);
            _paq.push(['setSiteId', '1']);
            var d = document,
                g = d.createElement('script'),
                s = d.getElementsByTagName('script')[0];
            g.type = 'text/javascript';
            g.async = true;
            g.defer = true;
            g.src = u + 'matomo.js';
            s.parentNode.insertBefore(g, s);
        })();
    </script>
    <!-- End Matomo Code -->
    ...
</head>
```

### Blade (For Laravel)
    1. 將先前獲得的網頁追蹤碼，加到blade檔的<head/>區塊，並確保全部blade檔都有包含該script
    2. 從cas取得工號及名稱，並加近追蹤碼
        需置於 _paq.push(['trackPageView']); 之前，不然吃不到參數

```script
_paq.push(['setUserId', {!! json_encode(cas()->user()) !!}]);
_paq.push(['setCustomDimension', 1, {!! json_encode(cas()->getAttribute('english_name')) !!}]);
_paq.push(['setCustomDimension', 2, {!! json_encode(cas()->user()) !!}]);
```

    3. 最終追蹤碼擺放大致如下:

```html
<head>
    ...
    <!-- Matomo -->
    <script type="text/javascript">
        var _paq = window._paq || [];
        /* tracker methods like "setCustomDimension" should be called before "trackPageView" */
        _paq.push(['setUserId', {!! json_encode(cas()->user()) !!}]);
        _paq.push(['setCustomDimension', 1, {!! json_encode(cas()->getAttribute('english_name')) !!}]);
        _paq.push(['setCustomDimension', 2, {!! json_encode(cas()->user()) !!}]);
        _paq.push(['trackPageView']);
        _paq.push(['enableLinkTracking']);
        (function() {
            var u = "//x.x.x.x:8080/";
            _paq.push(['setTrackerUrl', u + 'matomo.php']);
            _paq.push(['setSiteId', '1']);
            var d = document,
                g = d.createElement('script'),
                s = d.getElementsByTagName('script')[0];
            g.type = 'text/javascript';
            g.async = true;
            g.defer = true;
            g.src = u + 'matomo.js';
            s.parentNode.insertBefore(g, s);
        })();
    </script>
    <!-- End Matomo Code -->
    ...
</head>
```

### React (SPA)
    1. 安裝套件 
    
[@datapunt/matomo-tracker-react](https://www.npmjs.com/package/@datapunt/matomo-tracker-react/)

    2. 使用方法:
        參考 A27專案 hook寫法

- [A27 Lab Space & Rack Management / App.js](http://10.32.36.106:10080/TQMS/a27_lab_space_and_rack_management/a27_management/-/blob/master/resources/js/src/App.js)
- [A27 Lab Space & Rack Management / Default.js (Layout)](http://10.32.36.106:10080/TQMS/a27_lab_space_and_rack_management/a27_management/-/blob/master/resources/js/src/components/layouts/Default.js)
- [A27 Lab Space & Rack Management / use-matomo.js](http://10.32.36.106:10080/TQMS/a27_lab_space_and_rack_management/a27_management/-/blob/master/resources/js/src/hooks/use-matomo.js)


``` javascript
import React, { useEffect, createContext } from "react";
import { useSelector } from 'react-redux';
import { MatomoProvider as Provider, createInstance, useMatomo } from '@datapunt/matomo-tracker-react'

import useRouter from "./use-router";

const MatomoContext = createContext();

const MatomoProvider = ({ children }) => {
  const { user } = useSelector((state) => state.user)

  //urlBase及siteId於env管理
  const instance = createInstance({
    urlBase: process.env.MIX_MATOMO_URL_BASE,
    siteId: process.env.MIX_MATOMO_SITE_ID,
    userId: undefined, // optional, default value: `undefined`.
  })

  useEffect(() => {
    if (!user) return
    //使用者登入成功 於matomo紀錄使用者id及名稱
    //setCustomDimension為自定義參數用法
    instance.pushInstruction('setUserId', user?.emplid)
    instance.pushInstruction('setCustomDimension', 1, user?.name)
    instance.pushInstruction('setCustomDimension', 2, user?.emplid)
  }, [user])

  return (
    <MatomoContext.Provider>
      <Provider value={instance}>
        {children}
      </Provider>
    </MatomoContext.Provider>
  );
};

const useMatomoTrack = () => {
  const { trackPageView, trackEvent } = useMatomo()
  const { pathname } = useRouter();
  const { user } = useSelector((state) => state.user)

  //SPA需要自行處理觸發時機
  //監聽使用者及路徑，只要路徑變更即傳送網頁瀏覽事件
  useEffect(() => {
    if (!user) return
    trackPageView()
  }, [pathname, user])
}

export { MatomoProvider };
export default useMatomoTrack;
```
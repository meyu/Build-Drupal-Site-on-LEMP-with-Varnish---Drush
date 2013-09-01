所為何來
=
於 Ubuntu 上使用 Drupal 做為網站的內容管理系統，並使用 Drush 進行數項模組的安裝及設定。
  
本文的網頁伺服器為 Ubuntu + Nginx + MariaDB + PHP 的 LEMP 架構，並搭配 Varnish 進行網頁快取。  
詳細設定請參考[此文](https://github.com/meyu/Initial-Ubuntu-HTTP-Server)及[此文](https://github.com/meyu/Install-Nginx--MariaDB--PHP--Adminer-and-Varnish-on-Ubuntu--LEMP-Varnish-)。
  
適用環境
=
作業系統：Ubuntu Server 12.04  
網頁伺服器：Nginx 1.5.3  
資料庫系統：MariaDB 5.5.32  
指令碼語言：PHP 5.3.10  
資料庫管理介面：Adminer 3.7.1  
反向快取伺服器：Varnish 3.0.4
內容管理系統：Drupal 7.23 + Drush 6
加裝Drupal模組：Varnish HTTP Accelerator Integration 7.x-1.0-beta2、

安裝方式
=
###安
   
DONE.
<br>
<br>

補充說明
=
###本文網站設定資訊

* 網站資料：<code>/usr/share/nginx/www/</code>
* 管理者：<code>WebAdmin</code>
* 密碼：<code>web.password</code>
* 本機網址：[http://localhost/](http://localhost/)
* 本機測試：[http://localhost/info.php](http://localhost/info.php)

###本文資料庫設定

* 網頁管理介面入口：[http://localhost/adminer.php](http://localhost/adminer.php)
* 資料庫系統管理者 <code>root</code>
* 資料庫系統密碼 <code>root.password</code>

###相關設定檔
* PHP：<code>/etc/php5/fpm/php.ini</code> 及 <code>/etc/php5/fpm/pool.d/www.conf</code> 及 <code>/etc/php5/fpm/pool.d/user.conf</code>
* Nginx：<code>/etc/nginx/sites-available/default</code> 及 <code>/etc/nginx/nginx.conf</code>
* Varnish：<code>/etc/default/varnish</code> 及 <code>/etc/varnish/default.vcl</code>

###補強  

* 安全考量下，如您不需要透過網頁介面管理資料庫時，建議移除 Adminer：  
```bash
sudo rm /usr/share/nginx/www/adminer.php
```
* 如要測試 Varnish 運作情況，可使用以下指，並參考[此文](http://helpdesk.getpantheon.com/customer/portal/articles/425726)：  
```bash
curl -I http://dev.pantheon.gotpantheon.com/
```


參考資源
=
* [Installation on Ubuntu | Varnish Community](https://www.varnish-cache.org/installation/ubuntu)
* [Pantheon | Varnish caching for high performance](http://helpdesk.getpantheon.com/customer/portal/articles/425726)
* [Varnish HTTP Accelerator Integration | drupal.org](https://drupal.org/project/varnish)
* [Chaos tool suite (ctools) | drupal.org](https://drupal.org/project/ctools)
* [Views | drupal.org](https://drupal.org/project/views)
* [Views Slideshow | drupal.org](https://drupal.org/project/views_slideshow)
* [Flex Slider | drupal.org](https://drupal.org/project/flexslider)
* [FlexSlider Views Slideshow | drupal.org](https://drupal.org/project/flexslider_views_slideshow)
* [Media | drupal.org](https://drupal.org/project/media)
* [Libraries API | drupal.org](https://drupal.org/project/libraries)
* []()
* []()[]()

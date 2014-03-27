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
其它工具套件：PEAR 1.9.4 + Git 1.8.4 + Zip 3.0  
內容管理系統：Drupal 7.23 + Drush 6.0  
選用之 Drupal 樣版：Open Framework 7.x-2.04  
加裝之 Drupal 模組：Varnish HTTP Accelerator Integration 7.x-1.0-beta2、Chaos tool suite 7.x-1.3、Views 7.x-3.7、Views Slideshow 7.x-3.0、Flex Slider 7.x-2.0-alpha2、FlexSlider Views Slideshow 7.x-2.x-dev、Media、7.x-1.3、Libraries API 7.x-2.1、CKEditor - WYSIWYG HTML editor 7.x-1.13、CAPTCHA 7.x-1.0、Link 7.x-1.1、Email Field 7.x-1.2、Advanced help 7.x-1.0  
   
<br>
安裝方式
=
###前置作業
請使用具有 sudo 使用權的帳號登入。(本文的帳號為 <code>WebAdmin</code>)  
由於過程中將會使用到 Drush、Git 及 Zip 工具，請先完成相關工具之安裝：
```bash
sudo apt-get install php-pear && 
sudo pear channel-discover pear.drush.org && 
sudo apt-get install drush php5-gd zip && 
sudo pear install drush/drush && 
sudo aptitude install libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev build-essential && 
wget https://git-core.googlecode.com/files/git-1.8.4.tar.gz && 
tar -zxf git-1.8.4.tar.gz && mv git-1.8.4 git && cd git && 
make prefix=/usr/local all && sudo make prefix=/usr/local install && 
cd && rm -rf git git-1.8.4.tar.gz
```
詳細說明，可參見[此文](https://github.com/meyu/Install-Drush-on-Ubuntu)及[此文](https://github.com/meyu/Install-Git-on-Ubuntu)。  
<br>   

###本文之網站架設規劃
本文將以單一伺服器架設多網站為前提，規劃在網頁資料夾目錄(本文設定位置在 <code>/usr/share/nginx/www</code>)下放置多個網站目錄：web1、web2、web3...等，各資料夾做為將各網站之根目錄，並以其為代號；故資料夾將安放如<code>/usr/share/nginx/www/web1</code>、<code>/usr/share/nginx/www/web2</code>、<code>/usr/share/nginx/www/web2</code>...之樣貌。  
如只打算架構單一網站，則請直接將<code>網頁資料夾目錄</code>做為網站根目錄。  
  
以下將以 <code>web1</code> 網站之架設為例。
<br>

###安裝 Drupal
請前往網頁資料夾目錄，在下載 Drupal 後，將其置放為 web1 資料夾：(本文的 Drupal 版本為 7.23) 
```bash
cd /usr/share/nginx/www && 
drush dl drupal --drupal-project-rename=web1
```
接著，將設定 web1 這個網站。  
本文使用以下設定：
* 新增網站資料庫：<code>web1</code>
* 設定網站資料庫的管理者：<code>web1DataAdmin</code>
* 設定網站資料庫的管理者密碼：<code>web1DataAdmin.password</code>
* 設定網站的管理者：<code>web1Admin</code>
* 設定網站的管理者密碼：<code>web1Admin.password</code>
* 設定網站的管理者 Email：<code>web1Admin@example.com</code>

請留意網站與網站資料庫的差異，其帳號亦是分開設定。  
另，本文的資料庫系統管理者為<code>root</code>，密碼為 <code>root.password</code>。  

依上述設定，其指令如下：
```bash
cd web1/ &&
drush site-install \
--db-url=mysql://root:root.password@localhost/web1 \
--db-su=web1DataAdmin \
--db-su-pw=web1DataAdmin.password \
--site-name=web1 \
--account-name=web1Admin \
--account-pass=web1Admin.password \
--account-mail=web1Admin@example.com
```
注意，如資料庫系統內已有與 <code>web1</code> 同名之資料庫時，其內容將被清空。
<br>

###安裝及設定模組
本文預定使用以下模組：Varnish HTTP Accelerator Integration、Chaos tool suite、Views、Views Slideshow、Flex Slider、FlexSlider Views Slideshow、Media、Libraries API、CKEditor - WYSIWYG HTML editor、CAPTCHA、Link、Email Field、Advanced help。  
各模組之功能，可利用文末之參考資源，自行前往各模組之網頁了解。  
  
請輸入以下指令，以安裝並啟用以上所述之模組：
```drush
drush pm-download varnish ctools views views_slideshow flexslider flexslider_views_slideshow media libraries ckeditor captcha link email advanced_help &&
drush pm-enable varnish ctools views views_slideshow flexslider flexslider_views_slideshow media libraries ckeditor captcha link email advanced_help
```
建立模組外掛元件所需之資料夾，並移步至其中：
```bash
mkdir sites/all/libraries/ &&  
cd sites/all/libraries/
```
下載並放置 Views Slideshow 所需之 JQuery 元件：
```bash
git clone https://github.com/malsup/cycle.git && 
mv cycle jquery.cycle
```
下載並放置 Flex Slider 所需之 JavaScript 元件：
```bash
git clone https://github.com/woothemes/FlexSlider.git && 
mv FlexSlider flexslider
```
下載並放置 CKEditor 所需之元件，並設定 CKFinder 的 config.php 檔：  
(本文使用 CKEditor 4.2 及 CKFinder 2.4，欲選擇其它版本，請至 [CKEditor 官網](http://ckeditor.com/))
```bash
wget http://download.cksource.com/CKEditor/CKEditor/CKEditor%204.3.4/ckeditor_4.3.4_full.zip && 
wget http://download.cksource.com/CKFinder/CKFinder%20for%20PHP/2.4/ckfinder_php_2.4.zip && 
unzip ckeditor_4.3.4_full.zip && unzip ckfinder_php_2.4.zip && 
rm ckeditor_4.3.4_full.zip ckfinder_php_2.4.zip && 
nano ckfinder/config.php
```
找到 CheckAuthentication{} 段落，並將其註解或刪除。其內容看起來如：
```text
function CheckAuthentication()
{
//WARNING : DO NOT simply...
...
return false;
}
```
再找到<code>$baseDir = resolveUrl($baseUrl);</code>這行，於其下新增：
```text
require_once '../../../../includes/filemanager.config.php';
```
儲存後退出，再移動 CKFinder 的元件至 CKEditor 的元件資料夾內：
```bash
mv ckfinder/ ../modules/ckeditor/ckfinder
```













```bash
git clone https://github.com/douglascrockford/JSON-js.git &&
mv JSON-js json2
```

sudo nano ../../default/settings.php

UNCOMMENT # $cookie_domain = '...'; AND EDIT (AROUND LINE 338):
======================================================
$cookie_domain = 'localhost';



RUN CRON
drush core-cron

VIEW THE STATUS REPORT
drush core-requirements

CLEAR CACHE SPECIFIC OR ALL
drush cache-clear

EDIT SETTINGS OF php.ini OR settings.php OR .htaccess
drush core-config

UPDATE DRUPAL CORE & DATABASE
drush pm-update

UPDATE DRUSH
drush self-update




MODULES/THEME DOWNLOAD
drush pm-download

MODULES/THEME ENABLE 
drush pm-enable

MODULES/THEME DISABLE 
drush pm-disable

MODULES/THEME UNINSTALL 
drush pm-uninstall

MODULES LIST
drush pm-list
drush pm-list --no-core



USER ADD
drush user-create username --mail="email@email.com" --password="password"
USER PASSWORD CHANGE
drush user-password username --password="new_pass"
USER DELETE
drush user-cancel username



BACKUP CODES, FILES AND DATABASE
sudo drush archive-dump

RESTORE
sudo drush archive-restore

RSTYNC TO/FROM ANOTHER SERVER USING SSH
sudo drush core-rsync



REFERENCE
http://www.drupaltky.org/article/41
http://www.drush.org/
http://www.drush.org/#php-eval


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

* 


參考資源
=
###安裝  
* [Installation on Ubuntu | Varnish Community](https://www.varnish-cache.org/installation/ubuntu)
* [Pantheon | Varnish caching for high performance](http://helpdesk.getpantheon.com/customer/portal/articles/425726)

###模組  
* [Varnish HTTP Accelerator Integration | drupal.org](https://drupal.org/project/varnish)
* [Chaos tool suite (ctools) | drupal.org](https://drupal.org/project/ctools)
* [Views | drupal.org](https://drupal.org/project/views)
* [Views Slideshow | drupal.org](https://drupal.org/project/views_slideshow)
* [Flex Slider | drupal.org](https://drupal.org/project/flexslider)
* [FlexSlider Views Slideshow | drupal.org](https://drupal.org/project/flexslider_views_slideshow)
* [Media | drupal.org](https://drupal.org/project/media)
* [Libraries API | drupal.org](https://drupal.org/project/libraries)
* [CKEditor - WYSIWYG HTML editor | drupal.org](https://drupal.org/project/ckeditor)
* [CAPTCHA | drupal.org](https://drupal.org/project/captcha)
* [Link | drupal.org](https://drupal.org/project/link)
* [Advanced help | drupal.org](https://drupal.org/project/advanced_help)
* [Email Field | drupal.org](https://drupal.org/project/email)

###樣版  
* [Responsive Drupal Theme | Open Framework](https://openframework.stanford.edu/)
* [SU-SWS/open_framework](https://github.com/SU-SWS/open_framework)
* []()
* []()
* []()
* []()
* []()
* 

|원본주소 | https://httpd.apache.org/docs/2.4/upgrading.html |
|:----|:----|

# 아파치 2.2에서 2.4 업그레이드 가이드

업그레이드를 하는 기존 Apache HTTP Server 유저들을 위해 우리는 중요한 정보를 기술한 문서를 제공합니다.

이 문서는 간결하게 정리하였으며 자세한 내용은 [New Features](https://httpd.apache.org/docs/2.4/new_features_2_4.html)에서 확인 할 수 있습니다.

이 문서는 서버설정 변경이 필요하거나 2.2처럼 2.4를 사용하기 위한 방법을 기술합니다. 향상 된 2.4 버전의 기능을 사용하려면 New Features 문서를 확인하세요

이 문서는 오직 2.2에서 2.4의 변화만 기술합니다. 만약 2.0 버전에서 2.4로 업그레이드를 한다면, [2.0에서 2.2 업그레이드 가이드 문서](http://httpd.apache.org/docs/2.2/upgrading.html)를 봐야합니다.


## Compile-TIme Configuration Changes

컴파일 과정은 2.2와 유사합니다.당신의 이전 configure command line을(as found in build/config.nice in the installed server directory) 대부분 사용 할 수 있습니다. 아래에 몇가지 변경 된 기본세팅이 있습니다.

#### Some details of changes
* 제거 된 모듈: mod_authn_default, mod_authz_default, mod_mem_cache. If you were using mod_mem_cache in 2.2, look at mod_cache_disk in 2.4.

* All load balancing implementations have been moved to individual, self-contained mod_proxy submodules, e.g. mod_lbmethod_bybusyness. You might need to build and load any of these that your configuration uses.

* BeOS, TPF 와 더 오래 된 A/UX, Next, Tandem 같은 플랫폼의 지원을 종료하였습니다.

* 구성
  + dynamic modules (DSO)은 기본으로 탑재합니다.
  + 기본 모듈만 적재됩니다. 다른 LoadModule directives는 설정파일에서 주석처리 되었습니다.
  + the "most" module set gets built by default
  + the "reallyall" module set adds developer modules to the "all" set

## Run-Time Configuration Changes

authorization에 뚜렷한 변화가 있어서 2.4를 사용하기 전 2.2의 설정을 변경을 요구한다.

#### Authorization

authorization을 사용하려면 설정파일을 수정해야한다.

이 문서 [Authentication, Authorization and Access Control Howto](https://httpd.apache.org/docs/2.4/howto/auth.html)에서 새로운 authorization directives가 적용 되는 순서를 제어하기 위한 메커니즘을 설명 한 섹션[Beyond just authorization](https://httpd.apache.org/docs/2.4/howto/auth.html#beyond)을 확인 해야한다

 인증 된 유저와 일치 하지않을때 authorization modules이 응답하는 방법을 제어하는 Directives는 AuthzLDAPAuthoritative, AuthzDBDAuthoritative, AuthzDBMAuthoritative, AuthzGroupFileAuthoritative, AuthzUserAuthoritative, and AuthzOwnerAuthoritative 을 포함하여 제거되었습니다. 이러한 directive들은 더 잘 표현 된 [RequireAny](https://httpd.apache.org/docs/2.4/mod/mod_authz_core.html#requireany), [RequireNone](https://httpd.apache.org/docs/2.4/mod/mod_authz_core.html#requirenone), and [RequireAll](https://httpd.apache.org/docs/2.4/mod/mod_authz_core.html#requireall)로 대체되었습니다.

If you use [mod_authz_dbm](https://httpd.apache.org/docs/2.4/mod/mod_authz_dbm.html), you must port your configuration to use Require dbm-group ... in place of Require group ....

#### Access control

2.2에서의  client hostname, IP address, client requests 의 characteristics 를 기반한 access control을 위한 Order, Allow, Deny, and Satisfy를 사용한 directives는 사용을 하지않는다

2.4에서 그러한 access control은 새로운 모듈인 mod_authz_host를 사용한 authorization checks로 이뤄진다
이전 설정 호환을 위한 새로운 모듈 mod_access_compat이 제공 되더라도 이전 access control 지시문들은 새로운 인증 메커니즘에 따라 대체되어야 한다.

> #### 이전 과 현재 directive의 혼합사용
> 이전 Order, Allow, 또는  Deny 같은 directive와 새로운 Require같은 directive의 혼합사용은 기술적으로 가능하지만 권유하지는 않는다. [mod_access_compat](https://httpd.apache.org/docs/2.4/mod/mod_access_compat.html)은 설정에 이전 directive를 포함 가능하도록 지원하여 2.4로의 업그레이드를 용이하게 하려고 만들어졌다.

여기에 이전과 새로운 방식의 access control 예제가 있다.

이 예제는 인증없이 모든 request를 불허한다

>**2.2 configuration:**<br>
Order deny,allow<br>
Deny from all

>**2.4 configuration:**<br>
Require all denied

이 예제는 인증없이 모든 request를 허용한다

>**2.2 configuration:**<br>
Order allow,deny<br>
Allow from all<br>

>**2.4 configuration:**<br>
Require all granted

이 예제는 인증없이 example.org 도메인을 가진 host는 허용하도록한다; 모든 다른 host는 접근을 불허한다

>**2.2 configuration:**<br>
Order Deny,Allow<br>
Deny from all<br>
Allow from example.org

>**2.4 configuration:**<br>
Require host example.org

이 예제는 이전과 새 directive를 혼합하여 사용하여 예상치 못한 결과를 가져오는 예제이다.

>#### Mixing old and new directives: NOT WORKING AS EXPECTED
>DocumentRoot "/var/www/html"
>
><Directory "/"><br>
>&emsp;AllowOverride None<br>
>&emsp;Order deny,allow<br>
>&emsp;Deny from all<br>
>\</Directory><br>
>
><Location "/server-status"><br>
>&emsp;SetHandler server-status<br>
>&emsp;Require local<br>
>\</Location>
>
> access.log - GET /server-status 403 127.0.0.1<br>
> error.log - AH01797: client denied by server configuration: /var/www/html/server-status

왜 httpd는 접근을 허용하도록 설정을했는데 servers-status로의 접근을 거절했을까?
설정 시나리오에서 mod_access_compat(2.2지원) directives가 mod_authz_host(2.4 지)보다 우선순위가 있다.

거꾸로 아래의 예제는 예상대로 동작한다.

>#### Mixing old and new directives: WORKING AS EXPECTED<br>
>DocumentRoot "/var/www/html"
>
><Directory "/"><br>
>&emsp;AllowOverride None<br>
>&emsp;Require all denied<br>
>\</Directory>
>
>\<Location "/server-status"><br>
&emsp;SetHandler server-status<br>
&emsp;Order deny,allow<br>
&emsp;Deny from all<br>
&emsp;Allow From 127.0.0.1<br>
>\</Location> <br>
>
>access.log - GET /server-status 200 127.0.0.1

만약 혼합해서 사용이 가능하더라도 업그레이드 시에는 혼합하지 않는쪽으로 시도해보는것을 권유합니다.
이전 directive를 사용을 유지하고 이후 마이그레이션을 하던가 모두 마이그레이션 하든 한 쪽을 사용하는걸 유지하기바랍니다.


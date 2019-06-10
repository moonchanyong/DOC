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

 인증 된 유저와 일치 하지않을때 authorization modules이 응답하는 방법을 제어하는 Directives는 AuthzLDAPAuthoritative, AuthzDBDAuthoritative, AuthzDBMAuthoritative, AuthzGroupFileAuthoritative, AuthzUserAuthoritative, and AuthzOwnerAuthoritative 을 포함하여 제거되었습니다. 이러한 directive들은 더 잘 표현 된 RequireAny, RequireNone, and RequireAll로 대체되었습니다.


# SSO Client Package via Hwacom SSO

<a href="https://github.com/mozielin/Client-SSO/actions"><img src="https://github.com/mozielin/Client-SSO/workflows/PHP Composer/badge.svg" alt="Build Status"></a>
[![Total Downloads](http://poser.pugx.org/hwacom/client-sso/downloads)](https://packagist.org/packages/hwacom/client-sso)
[![Latest Stable Version](http://poser.pugx.org/hwacom/client-sso/v)](https://packagist.org/packages/hwacom/client-sso)
## 前言

因為Zizaco/entrust 套件已經停止維護了
但又好用到靠杯，所以暫時無法替換其他方案
只好接手繼續維護

## 安裝說明

```bash
composer require hwacom/entrust
```

## Service Provider設定 (Laravel 5.5^ 會自動掛載)

Composer安裝完後要需要修改 `config/app.php` 找到 providers 區域並添加:

```php
\Hwacom\SSO\SSOServiceProvider::class,
```

## Config設定檔發佈 

用下列指定會建立sso.php設定檔，需要在 `.env` 檔案中增加設定.

```bash
php artisan vendor:publish
```

 下列設定會自動增加在 `config/sso.php`

```php
'sso_enable'    => env('SSO_ENABLE',false),
'client_secret' => env("SSO_CLIENT_SECRET"),
'callback'      => env("SSO_CLIENT_CALLBACK"),
'sso_host'      => env("SSO_HOST")
```

在`.env` 中增加設定

```php
SSO_ENABLE          = true
SSO_HOST            = http://test.eip.hwacom.com:8000
SSO_CLIENT_SECRET   = ELg5TA5b5JTEJUCdDGoRo0mZIKQe1EuoF8W6ytvP
SSO_CLIENT_CALLBACK = http://test.crm.hwacom.com:8080/callback
```

## [LoginController] 增加兩個Function
Login

```
/**
 * 登入頁面置換，需自行寫入LoginController中
 * Laravel7 Function Name 改為 showLoginForm
 */
public function create()
{
    if (config('sso.sso_enable') === true ) {
        setcookie("callback", config('sso.callback'), 0, "/", '.hwacom.com');
        return redirect(config("sso.sso_host") .  "/google/auth");
    }
    return view('auth.login');
}
```

Logout

```
/**
 * 登出用需自行寫入LoginController中
 *
 * @param  \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\RedirectResponse|\Illuminate\Http\JsonResponse
 */
public function destroy(Request $request)
{
    if (config('sso.sso_enable') === true ) {
        setcookie("token", "", time() - 3600, '/', '.hwacom.com');
    }

    Auth::guard('web')->logout();

    $request->session()->invalidate();

    $request->session()->regenerateToken();

    return redirect(config("sso.sso_host"));
}
```
## [Middleware] 增加至`Http/Kernel.php`web Group中

```php
\Hwacom\ClientSso\Middleware\SSOAuthenticated::class,
```

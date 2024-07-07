        
<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

# auth-laravel-10-8-linkedin-login

- Linkedin Auth system
- Role based authentication system where we can give the user permission according to his role.
- Also see : [Laravel Socialite Documentation](https://laravel.com/docs/10.x/socialite)

### If downloading this
-------------------------------------------- START -----------------------------------------------<br> 
never forgot to run 
```sh
composer install
php artisan migrate     # never forgot to create database
composer require laravel/socialite
php artian serve
```
application is running on `http://localhost:8000`<br>

---------------------------------------------- END ------------------------------------------------<br> 

## Step 0
 - first you need to create a simple working login system 
 - you can take reference of : [auth-laravel-10-1](https://github.com/suraj-repositories/auth-laravel-10-1)

## Steps
1. require the 'socialite' package
```sh
composer require laravel/socialite
```

2. Now you need to create credentials on linkedinapp app
   - Go to [developer.linkedin.com](https://developer.linkedin.com/)
   - On the navbar `My apps` -> `Create app`
     * fill the details 
     * on the `LinkedIn Page` section you need to fill the company linkedin page url -> if you don't have it = create it.<br />
     To create it click on the link `Create a new LinkedIn Page`
     * fill other required details and create app
   - Your app will be created now 
   - go to `Products tab` and request access for `Share on LinkedIn` and `Sign In with LinkedIn using OpenID Connect`
   - After that go to the auth tab -> click on the link `OAuth 2.0 tools` and Create a new access token -> fill the details and click on `Request access token`
     * under the `Select scopes` choose `open id` and `email`
     * under `Confirm app settings changes:` - the input is auto generated - not change it here
     * tick the checkbox and click on `Request access token` <br />
     ( if error happen -  change your browser or private window and try again)
     * your token will be generated if not change your browser or private window and try again
   - After token generation go to `Auth` tab on your app 
     * create a new url for `Authorized redirect URLs for your app`
     * in my case `http://localhost:8000/auth/linkedin/callback` 
   - After that you can use your `Client ID` and `Primary Client Secret` 

- set those values in .env file
```sh
LINKEDIN_CLIENT_ID=**************
LINKEDIN_CLIENT_SECRET=****************************
LINKEDIN_REDIRECT=http://localhost:8000/auth/linkedin/callback
# or
LINKEDIN_REDIRECT=https://www.example.com/auth/linkedin/callback  # using on deployed website 

```
3. After the credential setup you need to open `config\services.php` in which you need to setup the following
```sh
   'linkedin-openid' => [
        'client_id' => env('LINKEDIN_CLIENT_ID'),
        'client_secret' => env('LINKEDIN_CLIENT_SECRET'),
        'redirect' => env('LINKEDIN_REDIRECT'),
    ],
```
4. on your login file create a link for login 
```php
 <a href="{{ url('/auth/linkedin') }}">Login with Linkedin</a>
```
5. setup the route for the URL and success-redirect-url
```php
    Route::get('/auth/linkedin', [AuthController::class, 'linkedinPage']);
    Route::get('/auth/linkedin/callback', [AuthController::class, 'linkedinRedirect']);
```
6. setup a controller method to handle linkedin login
```php
    public function linkedinPage(){
        return Socialite::driver('linkedin-openid')->redirect();
    }
```
```php
   public function linkedinRedirect(){
       try{
            $user = Socialite::driver('linkedin-openid')->user();
            $findUser = User::where('email', $user->email)->first();

            if(!$findUser){
                $findUser = new User();
                $findUser->email = $user->email;
                $findUser->password = Hash::make('123');
                $findUser->name = $user->name;
                $findUser->role = "USER";
                
                $findUser->save();
            }
           
            Auth::login($findUser);
            return redirect()->route('home');
            
       }catch(Exception $e){
            dd('Something went wrong!! : ' . $e->getMessage() );
       } 
    }
```
Now your app is ready to login with linkedin.

### Further steps
- Also when you make changes on env - do this command before test your application
```sh
php artisan cache:clear
php artisan optimize
php artisan serve
```

<br />
<p align="center">⭐️ Star my repositories if you find it helpful.</p>
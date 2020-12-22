# Basic login api examples

Here is a basic example. Where `loginapi.php`, can be called in two ways:

* `https://fgk.net/api/loginapi?req_typ=L&user_id=906203&user_pass=Piyush@123` Login get request api call url with necessary parameters. Instead `get_login()` function should be used.
```json
{"id":"906203","message":"User password matched","status":true,"db_status":"A","data":{"sec_cd":"51","name":"PIYUSH SACHAN"}}

{"id":"906203","message":"Incorrect login password..!","status":false,"db_status":"A","data":null}
```
* `https://fgk.net/api/loginapi?req_typ=R&user_id=906203&user_pass=Piyush@123&new_pass=Piyush@123` Password reset get request api call.

```json
{"id":"906203","message":"Password reset successfully","status":true,"db_status":"A"}
```

```php
<?php

/**
 * Login page
 * */
public function login()
{
    /** Site layout settings */
    $this->mainNav = TRUE;
    $this->title = 'FGK :: Login';
    $this->page = 'login';
    /** --------------------------- */

    $this->my_form_validation = TRUE;
    $this->load->library('form_validation');

    $usersList = implode(",", array_map(function($value){
                return $value->per_no;
        } , $this->HM->get_user())
    );

    $this->form_validation->set_rules(
        'user',
        'User',
        'trim|required|is_natural|max_length[6]|in_list[' . $usersList . ']',
        array(
            "max_length" => "%s can not be more than 6 characters",
            "in_list"  => "%s not exists !"
        )
    );

    $this->form_validation->set_rules('pass', 'Password', 'trim|required');

    if ($this->form_validation->run() !== FALSE) {
        $user = $this->input->post('user');
        $password = $this->input->post('pass');

        $response = $this->get_login($user, $password);

        if (!$response->status) {
            $this->flash_message('danger', $response->message, "times");
        } else {
            $this->flash_message('info', 'You logged in successfully !', "user");
        }

        if ($response->status === TRUE) {
            setcookie($this->cookieID, openssl_encrypt($this->secretKey . $user, 'des-ede3', $this->secretKey) , time() + (86400 * 1), "/"); // 86400 = 1 day
            $this->set_session_login($user);
        }
    }

    $this->siteLayout();
}

/**
 * Login request api
 * */
protected function get_login($user, $password, $type = "L")
{
    $domain = ENVIRONMENT === 'production' ? 'http://fgk.net' : 'http://devphp.fgk.net';
    $apiURL = "/api/loginapi";
    $apiParams = '?req_typ=' . $type . '&user_id=' . $user . '&user_pass=' . $password;

    $url = $domain . $apiURL . $apiParams;

    $ch = curl_init();                       // initialize CURL
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
    $output = curl_exec($ch);
    curl_close($ch);                         // Close CURL

    // Use file get contents when CURL is not installed on server.
    if(!$output){
        $output =  file_get_contents($url);
    }

    return json_decode($output);
}
```

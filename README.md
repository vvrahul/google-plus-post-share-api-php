# google-plus-post-share-api-php
Here is google Plus post share PHP api using Domain Api ,
session_start();
ini_set('display_errors', 1);
error_reporting(E_ALL);
$redirect_uri = 'http://' . $_SERVER['HTTP_HOST'] . $_SERVER['PHP_SELF'];
require_once 'vendor/autoload.php';
   
use Aws\S3\S3Client;
$client = new Google_Client();
$client->setClientId($client_id);
$client->setClientSecret($client_secret);
$client->setRedirectUri($redirect_uri);
$client->setAccessType("offline");
$client->setScopes(array(
    'https://www.googleapis.com/auth/plus.me',
    'https://www.googleapis.com/auth/plus.stream.write'   
));
if (isset($_REQUEST['logout'])) {
    unset($_SESSION['upload_token']);
}
if (isset($_GET['code'])) {
    $token = $client->fetchAccessTokenWithAuthCode($_GET['code']);
    $client->setAccessToken($token);
// store in the session also
    $_SESSION['upload_token'] = $token;
// redirect back to the example
    header('Location: ' . filter_var($redirect_uri, FILTER_SANITIZE_URL));
}
if (!empty($_SESSION['upload_token'])) {
    $client->setAccessToken($_SESSION['upload_token']);
    if ($client->isAccessTokenExpired()) {
        unset($_SESSION['upload_token']);
    }
} else {
    $authUrl = $client->createAuthUrl();
}
if ($client->getAccessToken()) {
    $t = $client->getAccessToken();
    if ($client->isAccessTokenExpired()) {
        $client->refreshToken($t['access_token']);
    }
##########################################################
    $accesstoken = $t['access_token'];
    $user_id = 'your google plus user id';
    $url = 'https://www.googleapis.com/plusDomains/v1/people/' . $user_id . '/activities';
    $headers = array(
        'Authorization : Bearer ' . $accesstoken,
        'Content-Type : application/json',
        
    );
    $post_data = array("object" => array("originalContent" => "Shareing Message to your account will be here"), "access" => array("items" => array(array("type" => "domain")),"domainRestricted" => true));
    $data_string = json_encode($post_data);
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "POST");
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $data_string);
    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_URL, $url);
    $file_result = curl_exec($ch);
    curl_close($ch);
    
    exit;

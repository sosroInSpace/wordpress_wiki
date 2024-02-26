Below is a script that can be used to send a list of the current instances plugins to a Google sheet.

- Make sure that the `range` variable is updated with the name of the sheet found here: 
https://docs.google.com/spreadsheets/d/18IBvZRe2XVPlQ3-Na_f9I2SmsknLXxtZvAoCkF8GZ8s/edit?usp=sharing

- Use a cronjob with `crontab -e` to run this script:
```
# Plugin list to google sheet (once monthly)
0 0 1 * * /opt/bitnami/php/bin/php /home/bitnami/scripts/plugin_scripts/get_plugins.php
```

### get_pluigins.php script
```
<?php

// TODO: validate input params, ignore bots...
require_once('/home/bitnami/apps/wordpress/htdocs/wp-load.php');
require_once('/home/bitnami/scripts/plugin_scripts/vendor/autoload.php'); // google-api-php-client path

$plugin_list = get_plugins();

$output = "";
foreach($plugin_list as $item) {
    $item = json_decode(json_encode($item), true);
    $name = $item["Name"];
    $version = $item["Version"];

    $output .= $name . " -- Version: " . $version . "\n";

}
echo $output;

function getClient()
{
    $client = new Google_Client();
    $client->setApplicationName('Project');
    $client->setScopes(Google_Service_Sheets::SPREADSHEETS);
    //PATH TO JSON FILE DOWNLOADED FROM GOOGLE CONSOLE FROM STEP 7
    $client->setAuthConfig('/home/bitnami/scripts/plugin_scripts/credentials.json'); 
    $client->setAccessType('offline');
    return $client;
}

// Get the API client and construct the service object.
$client = getClient();
$service = new Google_Service_Sheets($client);
$spreadsheetId = '18IBvZRe2XVPlQ3-Na_f9I2SmsknLXxtZvAoCkF8GZ8s'; // spreadsheet Id
$range = 'igawinthrop.com.au'; // Sheet name - UPDATE THIS ONLY - COINCIDE WITH SHEET NAME FROM SPREADSHEET

$valueRange= new Google_Service_Sheets_ValueRange();
$valueRange->setValues(["values" => ["a", "b"]]); // values for each cell
$valueRange->setValues(["values" => [
   $output
  , date("F j, Y, g:i a", time()) 
]]);

$conf = ["valueInputOption" => "RAW"];
$response = $service->spreadsheets_values->append($spreadsheetId, $range, $valueRange, $conf);


```

credentials.json
```
{
  "type": "service_account",
  "project_id": "polished-bridge-305205",
  "private_key_id": "f48db56fa8bb2481d675f7f5085c6cc54d166bef",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCmM9uoq4Sz+MZK\nufiX+rDa+Y8dXir8rZ9/WbvnAk/VooiQEwmPegsjj3POvlm70KXS8ostgFceB7vd\nmOwx7JFnvUPsG38cwWDnF9Of5+XaD6NuNzfEiRRZY3EU1VdQxDNYhhzYpNh0nOuw\niDtCNxTx/HCG7708+an+Gusq+3V8+IZEuzhhHbQ/O5Gm6PkY/GD6ku95UAPlZ/s1\nZH0FQEWIBuGOEs05bGgiLleqYzI+Ej2lNPfF2zlBCzWjdZJFf7e0Jz96KPlYFk3J\nVQPrQ68yW4Z0k+osDmUqE07S+/8gRr3O2zjitPNtdNI+P4YImihL9HWoalEPADCl\nr46HJNQLAgMBAAECggEAPNuhU75WbcYq6cL2NcRcjRoznufT6skyraGwjdLJY+tL\ngSIPbqOcP42wNKR73Ct3BOq+Ls+fVsYzMt3joxZCWg+yNtsMrP1cW9JcMeHqxvHS\nALIkcAlX07F3f07tVYw6VvBo0KVwAydQoEgKFuFvgHpUw/w2OYUcC4lU0lzYdQU9\n8tPYVfBCGfuwltULN71BQDwrRlJ2EFDVafZN5o5C9lN1M6FMKXnVakKIpKqLfznv\nxz+KxXXZlDQCaayltWv243fYavyvj4xCiy7HZz7Wtj00pryVlPp0wfauUVkL2tX0\n9lS7yUqDUl5PiZAoJL+MsuAhKg3NypVyTsQ46Hu/CQKBgQDjTfw+T1wucANrkW8G\nJFVxasmSgK2By7khhIRTz25IE7ruU+9DcOXoNuovIrkzzS1leOlLoUmOG9tM5Jz3\nbJHZJuD3Zen0IzZ0vmmHc/tVBZ+/2q5+3QBzvqgYKURQbQOHMF7LLagcst/UPZED\nBHJpVXbwzi7mfebx8+w4tzCzmQKBgQC7Ly5FowoW/s4gVBFxDnPQ3t4fYCWl/pPI\npAIJuG9QG3eJ0V7eiS99NrbzMbpNnqNYHtGbLlmenufSBsicGv4tVCrcevcQz4LW\n/rvb5k4QtzEEDbKdpj3nIaLB9QiRlXQVqG0j7xcgbFEB6I3vfhdbAFExpE/pJYGW\nbIYD1AhLQwKBgQCGf6/BQv49sCQl81Fppfg0+0Y4/REt21k5XwtZ+ES+O4aB3YKX\nOmegB1Z8+6Pw5fh1sZ0CFnbKsusJzcCfm0uV3a6CVXig8HEZlU4mS1etkH1dbc2Q\n3b6VvnwCh/CXUlojFVkSCnsOOD2/fYqf6XK1p0+Q37/avSb5hicBzEvyCQKBgGTI\nVGueCxKygp0ZZoKuu2Dcfk/6XorvdPZ0h7xgF17USxpjJmc/CdirGvn57ktYfK43\nebfJzur+t+Z3TI/wYKZbSPCJLHlaoSHM6azOZX3OhI+gKGmFVpMZox43JjPseiIn\nGwxb8OG+MCeM5M7r3vtaQl0uEPCBBzLMn6N0CDstAoGABN8zcW6Xj8KgLgWedRtk\n3K4s7YI0y3SsbXhmJ+fkGYMt8gpzGTLLkTFLejZSS2aESH0zOmMTi8MRTSEXVqDC\nJMIFN+0VAhyrcGvaFfk274COoAzJx0+DHsoatlxaEL4hwzTPeUH2XQrXdiOV5b5w\n8vHWTOXmmcHWMmoqOGHppsA=\n-----END PRIVATE KEY-----\n",
  "client_email": "service-plugin-list@polished-bridge-305205.iam.gserviceaccount.com",
  "client_id": "107855304215842093622",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/service-plugin-list%40polished-bridge-305205.iam.gserviceaccount.com"
}


```

Download composer in the current working directory via:

```
composer require google/apiclient:^2.0

```

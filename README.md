# ZarrinPal gateway using CodeIgniter
It's about how to setup a gateway via CodeIgniter library

This article has been made by <a href="http://graphap.com">Graphap</a>

You can find this article in persian language in <a href="http://graphap.com">Graphap</a>

So I want to explain what we need at the first time and then explore to each step :

1 - A file which has some HTML code for user to choose his/her plan and then clicked on payment button<br />
2 - I have used ajax for send and receive data, So we need a file which have some Javascript codes<br />
3 - After that we create some codes within a controller<br />
4 - And the last section we create a library to do what ever we want for payment operation<br />

Let’s begin :

first section, I create and HTML file into View for my users :

```html
<div class="row">
  <div class="col-xs-12">
    <div class="col-xs-12 form-group">
      <label for="">مدت زمان : </label>
    </div>
    <div class="col-xs-12 form-group">
      <select class="" id="choose_plan">
        <option>1 ماه</option>
        <option>2 ماه</option>
        <option>3 ماه</option>
      </select>
    </div>
  </div>
  <div class="col-xs-12">
    <button type="submit" class="btn btn-default payment">پرداخت</button>
  </div>
</div>
```
As simple as you see, I created HTML code and now User can choose the plan.
Second section, I pass my parameters via Ajax to my Controller :
```
$(document).ready(function() {
  $('body').on('click', '.payment', function ()
  {
    $.ajax({
      type	: "POST",
      dataType: "json",
      url		: "http://example.com/vitrin_payment", 
      cache   : false,
      data	:
      // Sent parameter for the request
      {
                my_plan: $('#choose_plan :selected').val()
      },
      success: function(data)
            {
              if ( data.au != '' ) {
                  window.location.replace("https://sandbox.zarinpal.com/pg/StartPay/"+data.au); // I'll explain this line in the last section.
              }
      }
    });
  });
});
```
So We're ready to create our Controller into Controller folder and named that My_Controller.php :
```
<?php
  defined('BASEPATH') OR exit('No direct script access allowed');
  class My_Controller extends CI_Controller {
    public function vitrin_payment()
    {
      $users_id = $this->session->userdata('user_id'); // We need get user id from session
      $plan = $this->input->post('my_plan',true); // We get value from Ajax
      if ( $plan != NULL ) {
        // Check for the plan
        $get_amount = $this->ZarrinPal_Model->check_vit_value($plan); // In this section i send the plan to ZarrinPal Model to check that plan is true or someone has another idea! (I'll show you the Model)
        // Set properties
        $description = 'پرداخت برای خرید'; // We need to set a small description about payment operation
        $callbackurl = 'http://example.com/My_Controller/verify_vitrin/'.$get_amount.'/'.$users_id; // We need this callback url to verify user purchase
        if ( $get_amount ) { // So, if that plan exist we're going to catch money from user!
          // Load ZarrinPal library that i will show you how to create this, We need to load the library
          $this->load->library('ZarrinPal');
          // We pass these necessary parameters to begin payment operation
          $this->zarrinpal->do_payment($users_id,$plan,$get_amount,$description,$callbackurl);
        } else {
          // If the plan was wrong show this error message using Json
          echo json_encode(array(
            'alert_style'	=> 'alert-danger',
            'alert_text'	=> 'مقدار فرستاده شده برای پرداخت نامعتبر است!'
          ));
        }
      } else {
        // We check the plan was successfuly send to our Controller and if it was empty show this error message
        echo json_encode(array(
            'alert_style' => 'alert-danger',
            'alert_text' => "مقداری وارد نشده است!"
          ));
      }
    }
  }
?>
```

It has continue

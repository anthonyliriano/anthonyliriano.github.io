---
layout: default
title: Contact
---
<section class="site-header">
    <h1 class="smallcap"><a class="site-title" href="{{ '/' | prepend: site.baseurl | prepend: site.url }}">
        Anthony Liriano
    </a></h1>
    {% include nav.html %}
</section>
<div id ="contact-form">
<h2>Get In Touch</h2>
<p>Please feel free to contact me via the form below.</p>

<div id="repos">
    <div class="container-fluid">
        <div class="row">
            <form class="row needs-validation" novalidate>
                <div class="col-lg-5 col-sm-12">
                    <label for="email" class="form-label">E-mail</label>
                    <input type="email" class="form-control" id="email" required>
                    <label for="message" class="form-label mt-2">Message</label>
                    <textarea class="form-control" id="message" rows="3" required ></textarea>
                    <div class="d-grid gap-2">
                        <button class="btn btn-dark btn-block mt-2" type="submit" onclick="UserAction()">Send Message</button>
                    </div>
                </div>
            </form>
        </div>
    </div>
</div>
</div>

<script>
    function UserAction() {
    console.log('sending http request');
    var xhttp = new XMLHttpRequest();
    xhttp.onreadystatechange = function() {
         if (this.readyState == 4 && this.status == 200) {
             alert(this.responseText);
         }
    };

    var email = document.getElementById('email').value;
    var message = document.getElementById('message').value;
    const json =         
    {
      "type" : "contact_form",
      "source" : "anthonyliriano.com",
      "ip": "-1",
      "sendToEmail": ["anthonylir@gmail.com"],
      "sendBccTo": [],
      "contact": {
        "firstName" : "",
        "email" : "",
        "phone" : "",
        "company": ""
      },
      "form" : {
       "isSensitive" : false,
       "fields" : {
           "message" : ""
       }
      }
    };

    xhttp.open("POST", "https://notify.lirilabs.com/api/v1/notify?clients=slack,email", true);
    xhttp.setRequestHeader("Content-type", "application/json");
    // xhttp.send(JSON.stringify(json));
}
</script>

<script>
    // Example starter JavaScript for disabling form submissions if there are invalid fields
(function () {
  'use strict'

  // Fetch all the forms we want to apply custom Bootstrap validation styles to
  var forms = document.querySelectorAll('.needs-validation')

  // Loop over them and prevent submission
  Array.prototype.slice.call(forms)
    .forEach(function (form) {
      form.addEventListener('submit', function (event) {
        if (!form.checkValidity()) {
          event.preventDefault()
          event.stopPropagation()
        }

        form.classList.add('was-validated')
      }, false)
    })
})()
</script>
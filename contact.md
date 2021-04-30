---
layout: default
title: Contact
---

## Get In Touch
Please feel free to contact me via the form below.

<div id="repos">
    <div class="container-fluid">
        <div class="row">
            <div class="col-lg-4 col-sm-12">
              <div class="mb-3">
                <label for="email" class="form-label">Email address</label>
                <input type="email" class="form-control" id="email" placeholder="name@example.com">
            </div>
            <div class="mb-3">
            <label for="message" class="form-label">Message</label>
            <textarea class="form-control" id="message" rows="3"></textarea>
            </div>
            <div class="d-grid gap-2">
                <button class="btn btn-dark" type="button" onclick="UserAction()">Send Message</button>
            </div>
            </div>
        </div>
    </div>
</div>

<script>
    function UserAction() {
    console.log('sending http request')
    var xhttp = new XMLHttpRequest();
    xhttp.onreadystatechange = function() {
         if (this.readyState == 4 && this.status == 200) {
             alert(this.responseText);
         }
    };
    xhttp.open("POST", "Your Rest URL Here", true);
    xhttp.setRequestHeader("Content-type", "application/json");
    xhttp.send("Your JSON Data Here");
}
</script>

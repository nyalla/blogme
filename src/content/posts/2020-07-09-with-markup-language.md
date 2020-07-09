---
template: blog-post
title: Setup a personal website/blog with Github pages
slug: setup-a-personal-website-with-Github-pages
date: 2020-07-09 14:22
description: Setup a personal website/blog with Github pages
---

Nowadays website for personal and portfolio became quite common. Its became a digital identity in the digital world. So getting up a personal website its a not big deal when we have numerous ready-made templates available over the internet with free of cost.
In this article, we are using [Github pages](https://pages.github.com/) to build a static website. Github pages provide us free hosting and no need to pay anything. And it is a free code repository for private and public repositories.
All you need to do is get the website template based on the available free themes and host them on Github.

For this article, I am using this [template](https://startbootstrap.com/themes/clean-blog/)  which is free. You can select any kind of template from the site.

[Download](https://github.com/BlackrockDigital/startbootstrap-clean-blog/archive/gh-pages.zip)  template to your local system. This contains all the HTML code necessary for a website. This all code is static which means there is no server-side code like PHP etc.

Now open an account in Github if not there earlier.

Then create a public repository with name as `<github username>.github.io`
Now extract the ZIP file from the downloaded theme.
Upload all the code in that ZIP file folder to the created public repository. (Drag all the content of all the files into the Github repository ).

Now go to the repository and check index file is available.
Now click on **Settings tab** of the repository. Scroll down to the **GitHub pages** section and selects **master**  from the dropdown **Source**  . After you selected master from the dropdown you will find the link above , which is the link for your website. Its something like `https://<github username>.github.io/`

If you click on that link you can navigate to the website which you created just now with Github pages and free themes.

To customize it further you can edit the HTML files and keep your names and further modifications.

We can also directly generate the websites based on the Jekyll themes from the GitHub pages section.

To make it more dynamic like whenever any person trying to contact through **Contact**  page, we can route that to any API or Google spreadsheet.

Here I tried routing contact page message to Telegram bot where I can get a message to my Telegram messenger.

You can find [here](https://nyalla.github.io/post1.html)  how to create a telegram bot. To make it simple, its nothing but a URL we can get to post messages.

URL looks like `https://api.telegram.org/bot{bot_token}/sendMessage?chat_id=123456789&text= {any text message}`

After this, we have to make changes in our code. We need to edit the file **contact_me.js**  which is inside js folder of our repository.

Make below changes to contact_me.js

    submitSuccess: function($form, event) {
          event.preventDefault(); // prevent default submit behaviour
          // get values from FORM
          var name = $("input#name").val();
          var email = $("input#email").val();
          var phone = $("input#phone").val();
          var message = $("textarea#message").val();
    	  var parmaDetails = "chat_id={chat_id}&text=";
    	  var payload= parmaDetails.concat(" Name:",name," Email:",email," Phone no:",phone," Message:",message,"&parse_mode=HTML");
    	  console.log(payload);
          var firstName = name; // For Success/Failure Message
          // Check for white space in name for Success/Fail message
          if (firstName.indexOf(' ') >= 0) {
            firstName = name.split(' ').slice(0, -1).join(' ');
          }
          $this = $("#sendMessageButton");
    	  //alert("hii");
          $this.prop("disabled", true); // Disable submit button until AJAX call is complete to prevent duplicate messages
          $.ajax({
            url: "https://api.telegram.org/bot503186988:{api-token}/sendMessage",
            type: "GET",
            data: payload,
            cache: false,
            success: function() {
    			console.log("inside success");
              // Success message
              $('#success').html("<div class='alert alert-success'>");
              $('#success > .alert-success').html("<button type='button' class='close' data-dismiss='alert' aria-hidden='true'>&times;")
                .append("</button>");
              $('#success > .alert-success')
                .append("<strong>"+"Hello  " + firstName + ", I will get back to you shortly. </strong>");
              $('#success > .alert-success')
                .append('</div>');
              //clear all fields
              $('#contactForm').trigger("reset");
    			var msg = new SpeechSynthesisUtterance();
    			msg.text = firstName.concat(", Thanks for submitting");
    			window.speechSynthesis.speak(msg);
            },
            error: function() {
    			console.log("inside error");
              // Fail message
              $('#success').html("<div class='alert alert-danger'>");
              $('#success > .alert-danger').html("<button type='button' class='close' data-dismiss='alert' aria-hidden='true'>&times;")
                .append("</button>");
              $('#success > .alert-danger').append($("<strong>").text("Sorry " + firstName + ", it seems that my mail server is not responding. Please try again later!"));
              $('#success > .alert-danger').append('</div>');
              //clear all fields
              $('#contactForm').trigger("reset");
            },
            complete: function() {
              setTimeout(function() {
                $this.prop("disabled", false); // Re-enable submit button when AJAX call is complete
              }, 1000);
            }
          });

We are replacing POST method with GET method and sending payload with all the parameters from HTML form.

It is a simple blog with minimal changes to the code. So whenever any new blog is written, you can make a new post.html file and have to link it from the index file.
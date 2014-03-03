Say we want to modify h1 element
Javascript interact w/DOM
Browser builds node model
Every browser has a slightly different DOM interpretation
That's where jQuery comes in! (Unless you listen to angry people on r/programming)

jQuery(code);

How can we find h1?
jQuery("h1");
$("h1");

How do we modify?
$("h1").text("New text!");

But the JS might fire before the DOM loads!
jQuery(document).ready(function(){
  $("h1").text("New text!");
});

Selected outputs for reference:
$(document).ready(function(){
  $("span").text("$100");
});

Using jQuery:
1. Download
2. <script src="jquery.min.js"></script>
3. <script src-"application.js"></script>

Write in application.js

Changing multiple elements at once. e.g. <li>
$("li");
$("li").text("Orlando");

How do we select one? ID or class
$("#container")
$(".articles");

Basic descendant selector
$("#destinations li");

What if you want only the direct descendent? Child selector
$("#destinations > li");

Select multiple elements
$(".promo, #france");

Select first item in list: pseudoselector
$("#destinations li:first");
$("#destinations li:last");
$("#destinations li:odd"); //index starts at zero

Combo from exercises
$("#destinations > li:even");

Part 2: Traversing DOM
What if we don't want to use a descendant selector
$("#destinations").find("li"); //takes a bit more code, but faster
$("li").first();
$("li").first().next(); // "method chaining"
$("li").first().next().prev(); // back to first
$("li").last();

Traverse up the DOM, child to parent
$("li").first().parent();

Walk down the DOM
$("#destinations").children("li"); //direct descendants, unlike find()

Part 3: Manipulating the DOM
Append a new DOM node, remove a button
$(document).ready(function() {
  //create a <p> node with the price
  var price = $('<p>From $399.99</p>');
  $('.vacation').append(price); //prepend also
  price.appendTo($('.vacation'));
  $('button').remove();
});

methods: .appendTo, .prependTo, .insertAfter, .insertBefore

exercises
message.prependTo($("book.button"));
$("li.usa").append(message);
$(".book").remove();

Acting on interaction
$(document).ready(function() {
  $('button').on('click', function() {
    var price = $('<p>From $399.99</p>');
    price.appendTo($('.vacation'));
    $('button').remove();
  )};
});

Key word: "this" - item that triggered the event.
$(this).remove //replaces $('button').remove();
$(this).after(price);
$(this).parent().parent().append(price); //breakable
$(this).parents('vacation').append(price); //ditto
$(this).closest('.vacation').append(price); //walk up to find ancestor

$(document).ready(function(){
  $('button').on('click', function(){
    var message = $("<span>Call 1-555-jquery-air to book this tour</span>");
    $(this).closest("li").append(message);
    $(this).remove();
  });
});

$(document).ready(function(){
  $(".tour").on("click", function(){
    var message = $("<span>Call 1-555-jquery-air to book this tour</span>");
    $(this).append(message);
    $(this).find("button").remove();
  });
});

How do we give them different prices?
data- tag <!-- HTML 5 attribute -->
<li class="vacation onsale" data-price="399.99">

$('.vacation').first().data('price');
"399.99"

var amount = $(this).closest('.vacation').data('price');
var price = $('<p>From $'+amount+'</p>')
var vacation = $(this).closest('.vacation'); //DRY, only query DOM once

('.vacation').on('click', 'button', function() {}) // "event delegation"

Filtering HTML
<div id="filters">

$('#filters').on('click', '.onsale-filter', function() {
  $('.highlighted').removeClass('highlighted');
  // find all vacations on sale
  // add class to those vacations
  $('.vacation').filter('.onsale').addClass('highlighted'); //removeClass
});

$(document).ready(function(){
  $("button").on("click", function(){
    var discount = $(this).closest(".tour").data("discount");
    var message = $("<span>Call 1-555-jquery-air for a $<dicsount>"+discount+"</discount> discount.</span>");
    $(this).closest(".tour").append(message);
    $(this).remove();
  });
});

$(document).ready(function(){
  $("button").on("click", function(){
    var tour = $(this).closest('.tour');
    var discount = tour.data("discount");
    var message = $("<span>Call 1-555-jquery-air for a $" + discount + "</span>");
    tour.append(message);
    $(this).remove();
  });
});

$(document).ready(function(){
  $(".tour").on("click", "button", function(){
    var tour = $(this).closest(".tour");
    var discount = tour.data("discount");
    var message = $("<span>Call 1-555-jquery-air for a $" + discount + " discount.</span>");
    tour.append(message);
    $(this).remove();
  });
});

$(document).ready(function(){
  $("#filters").on("click", ".on-sale", function() {
    $('.tour').filter('.on-sale').addClass('highlight');
  });
});

$(document).ready(function(){
  $("#filters").on("click", ".on-sale", function(){
    $('.highlight').removeClass('highlight');
    $(".tour").filter(".on-sale").addClass("highlight");
  });

  $("#filters").on("click", ".featured", function(){
    $('.highlight').removeClass('highlight');
    $(".tour").filter(".featured").addClass("highlight");
  });
});

Part 4: listening to DOM events
<li class="confirmation">
  <button>FLIGHT DETAILS</button>
  <ul class="ticket">...</ul>
</li>

.ticket {
  display: none;
}

- watch for click
- find ticket
- show it

$('.confirmation').on('click', 'button', function() {
  $(this).closest('.confirmation').find('.ticket').slideDown();
});

(slideUp, slideToggle)
debugging
alert('Hello');
$('li').length; //returns number of nodes on page
alert($('button').length);

$(document).ready(function () {
  alert($('img').length);
});

mouse events:
click, dblclick, focusin, focusout, mousedown, mouseup, mouseenter, mouseleave, mouseover, mousemove, mouseout

extract out
function showTicket() {
  $(this).closest('.confirmation').find('.ticket').slideDown();
}

$(document).ready(function() {
  $('.confirmation').on('click', 'button', showTicket);
  $('.confirmation').on('mouseenter', 'h3', showTicket);
}) // don't add parentheses at the end - will call function immediately

function showPhotos() {
  $(this).find("span").slideToggle();
}

$(document).ready(function() { 
  $("#tour").on("click", "button", function() { 
    $(".photos").slideToggle();
  });
  $(".photos").on("mouseenter", "li", showPhotos)
    .on("mouseleave", "li", showPhotos);
});

Keyboard events
<div class="vacation" data-price='399.99'>
  <h3>Hawaiian Vacation</h3>
  <p>$399.99 per ticket</p>
  <p>
    Tickets:
    <input type='number' class='quantity' value='1' />
  </p>
</div>
<p>Total Price: $<span id='total'>399.99</span></p>

Update input, calculate total price

$(document).ready(function() {
  $('.vacation').on('keyup' , '.quantity', function() {
    // Get the price for this vacation
    var price = +$(this).closest('.vacation').data('price')
    // Get the quantity entered
    var quantity = +$(this).val();
    // Set the total to price * quantity
    $('#total').text(price * quantity);
  });
});
Keyboard events: keypress, keydown, keyup
Form events: blur, select, change, focus, submit
Here we listen for keyup
Add a plus to the beginning of statment convert string to float (bleah
Use .val() to get value

$(document).ready(function() {
  $("#nights").on("keyup", function() {
    $("#nights-count").text($(this).val());
    $("#total").text(+$(this).val() * $(this).closest('.tour').data('daily-price'));
  });
});

$(document).ready(function() {
  $("#nights").on("keyup", function() {
    var nights = +$(this).val();
    var dailyPrice = +$(this).closest(".tour").data("daily-price");
    $("#total").text(nights * dailyPrice);
    $("#nights-count").text($(this).val());
  });
  $('#nights').on("focus", function() {
    $(this).val(7);
  });
});

Links and fade in

.comments {
  display: none;
}

// find the comments ul
$(this).closest('.vacation').find('.comments').fadeToggle();
// show the comments ul
fadeIn(), fadeOut(), fadeToggle()

But the page jumps to the top! Annoying! Why?
Click event bubbles up to each parent node. When it hits the top, browser moves there

function(event) {
  event.stopPropagation(); // nope!
  event.preventDefault(); // prevent default behavior, popping to top of page.
}

$(document).ready(function() {
  $(".see-photos").on("click", function(event) {
    event.stopPropagation();
    event.preventDefault();
    $(this).closest(".tour").find(".photos").slideToggle();
  });
  $(".tour").on("click", function() {
    alert("This should not be called");
  });
});

Section 5: styling

$(document).ready(function() {
  $('#vacations').on('click', '.vacation', function() {
    $(this).css('background-color', '#252b30')
    .css('border-color', '1px solid #967');
  });
});

.css(<attr>, value), .css(<attr>), .css(<object>)

$(this).css({'background-color': '#252b30',
             'border-color': '1px solid #967'});

$(this).find('.price').css('display', 'block'); //ok
$(this).find('.price').show(); //better

best, move the CSS back into application.css file:
.highlighted {
  background-color: #252b30;
  border-color: 1px solid #967;
}

.highlighted .price {
  display: block;
}

$(this).addClass('highlighted');
$(this).toggleClass('highlighted');

$(document).ready(function() {
  $(".tour").on("mouseenter", function() {
    $(this).css({"background-color": "#252b30", "font-weight": "bold"});
    $(this).closest(".tour").find(".photos").show();
  });
});

Animation
$(this).css({'top': '-10px'}); // not best
$(this).animate({'top': '-10px'});

if($(this).hasClass('highlighted')) {
  $(this).animate({'top': '-10px'});
} else {
  $(this).animate({'top': '0px'});
}

$(this).animate({'top': '-10px'}, 400); //400 ms - default
"fast" 200 ms
"slow" 600 ms
(also: slideToggle, fadeToggle, slideUp, slideDown, etc.)

//warning, only works with browsers that support CSS transitions. blah.
.vacation {
  transition: top 0.2s;
}

.highlighted {
  top: -10px;
}

$(document).ready(function() {
  $(".tour").on("mouseenter", function() {
    $(this).addClass("highlight");
    $(this).find(".per-night").animate({"top": "-14px","opacity": "1"}, "fast");
  });
  $(".tour").on("mouseleave", function() {
    $(this).removeClass("highlight");
    $(this).find(".per-night").animate({"top": "0px","opacity": "0"}, "fast");
  });
});

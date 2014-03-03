Haml is a more succinct version of erb.
HTML abstraction markup language.
Haml is a drop-in replacement for erb.
Install the haml gem, change the extension to .haml, and that's it.

erb <strong><%= item.title %></strong>
haml %strong = item.title

erb <strong class="code" id="message">Hello, World!</strong>
haml %strong{:class => "code", :id => "message"} Hello, World!

ID and class have special shorteners:
%strong.code#message Hello, World!

Don't even need to say div.
HTML: <div class="content">Hello, World!</div>
Haml: .content Hello, World!

erb 
<div class="item" id='item<%= item.id %>'>
	<%= item.body %>
</div>

haml
.item{:id => "item#{item.id}"}= item.body

erb 
<div id='content'>
  <div class='left column'>
    <h2>Welcome to our site!</h2>
    <p><%= print_information %></p>
  </div>
  <div class='right column'>
    <%= render :partial => "sidebar" %>
  </div>
</div>

haml
#content
  .left.column
    %h2 Welcome to our site!
    %p= print_information
  .right.column
    = render :partial => "sidebar"

Notes from docs
  - whitespace active
  - escapes html by default
  - you can include some html tags, and they will be passed through unmodified
  - use backslash to escape the first character of a line, rendering as plaintext.
  - forward slash at the end of a line renders without a closing tag, or with a self-closing tag in :xhtml.
  - using < removes all whitespace within a tag
  - using > removes all whitespace surrounding a tag
  - !!!XML - generate XML header
  - !!! - generate HTML header
  - forward slash at the beginning of the line to comment
  - wrap comment around several lines by indenting them inside a /
  - /[if IE] - Internet Explorer conditional comments
  - -# is a silent comment that doesn't get translated into html (including nested text)
  - insert Ruby with =
  - this will sanitize any HTML-sensitive characters generated, assuming :escape_html is set
  - do not nest code inside tag *ending* with =
  - you use - to proceed silently evaluated Ruby code... generally should put this in controllers, etc. though.
  - note that Ruby blocks do not need to be explicitly closed in Haml.
  - ~ runs like =, except it runs the Haml helper find_and_preserve on input
  - you can also interpolate without using = at the start of the line, just use #{}. Use \ to escape #{}, but note that it does not act as an escape anywhere else
  - &= escapes HTML
  - != unescapes HTML
  - : designates a filter 
  - | multiline code, e.g. long string
  - see faq for precise whitespace manipulation

convert to html:
haml hamlfile.haml output.html
---
title: ""
date: 2019-08-04T09:18:12-04:00
draft: False
---
## Anselus Text Markup (AnTM)

**Status:** Review  
**Submission Date:** August 8, 2019  
**Submitted By:** Jon Yoder <jsyoder@mailfence.com>  
**Abstract:** Rich text formatting language for client-side use  

### Changelog

0.0.1: Initial submission

### Description

AnTM, pronounced *"AN-tim"* or *"an-tee-EM"*, is a plaintext-with-markup format for describing rich formatting in a way that is expressive, easy to parse, and easy to implement with a measure of safety and security. It borrows heavily from BBCode, but introduces changes to make it more consistent and full-featured while retaining simplicity. It is intended to be the transmission format for all text on the Anselus platform, including messages, notes, and so on.

Anselus clients have a need to provide users the ability to communicate in an expressive manner in the same way that has been established with HTML e-mail. HTML, unfortunately, presents a number of challenges, including security and historical quirks in its syntax.

The problems which AnTM is intended to solve in replacing HTML for rich formatting are as follows:

- *Unnecessary Complexity* - No CSS. Simpler syntax. Easier parsing.
- *Text Client Compatibility* - HTML is notoriously bad for e-mail clients like Pine. Mailing lists should have a much easier time compiling digests.
- *Accessibility Problems* - The client has final say on how a message is displayed and easier parsing gives control to accessibility tools like screen readers.
- *Security* - no embedded scripting. Period.
- *Bloat* - Messages are much smaller because of fewer header fields, simpler markup, and messages don't need to internally store two different copies of the same content. A web browser is not required to be a component of a messaging client, reducing binary size and code complexity.	
- *Privacy* - Web beacons/bugs are eliminated because messages are self-contained. Read receipts require the request of the sender and permission of the recipient.

### Syntax

AnTM marks text with pairs of tags enclosed in square brackets. Each opening tag MUST have a corresponding closing tag. Each closing tag MUST begin with a forward slash (/) immediately preceding the text of the closing tag. White space MAY be found between the square brackets and the enclosed text, but space MUST NOT be found between the slash and the tag text. For example:

**Correct**  

<pre>
[b]Bold text[/b]
[ b ]Bold text[ /b ]
[b ]Bold text[ /b]
</pre>

**Incorrect**  

<pre>
[b]Bold text[\b]
[b]Bold text[/ b]
[b]Bold text[b/]
</pre>

Opening tags MAY have parameters, depending on the specific tag. These parameters MUST follow the tag name and MUST be separated by one or more spaces or tabs. Newlines and carriage returns MUST NOT be placed within the tags.

Each parameter MUST follow the format of parameter name, equal sign, and value enclosed by double quotes. If a double quotation mark needs to be part of the parameter value, the escape code *\u0034* MUST used in its place. Other escape codes MAY be used to specify other Unicode characters using the same format.

**Correct**  

<pre>
[font family="Arial" size="12" color="blue"]Styled text[/font]
[tag param1="Being \u0034cute\u0034"]Tag text[/tag]
</pre>

**Incorrect**  

<pre>
[font family='Helvetica' size='9' color='red']Styled text[/font]
[font family:"Helvetica" size:"9" color:"red"]Styled text[/font]
</pre>

### Embedded Content

As a security and privacy measure, all content referenced in a document MUST be local to the client, such as a file or database record. Only filenames are used. The exact method of storing and retrieving said content is left to the implementor. For example, in the case of a note in which the user has inserted an image, the image MUST be local to the PC on which the note is written. Images obtained from the World Wide Web MAY be used ONLY if the image is first downloaded to local storage and then a link created which points to the downloaded copy.

Any references to non-local content SHOULD be displayed clearly as a hyperlink and be in a way that the user is able to see the exact destination. URL de-obfuscation is not a requirement, but is highly recommended. Conversion of escape codes to actual characters, such as %20 for spaces, is recommended for link display to aid less-technical users in understanding link destinations. Although not required, integration of website/domain reputation services is also recommended for client implementations.

### Tag Reference

<a name="align"></a>
#### Align Text

**Syntax**  

`[align type="left"]aligned text[/align]`

**Parameters**

*type*: REQUIRED. Specifies the type of alignment. May be `left`, `center`, `right`, or `justified`.

**Notes**

This tag specifies horizontal alignment of enclosed text.

<a name="bold"></a>
#### Bold

**Syntax**  

`[b]Bold text[/b]`

**Parameters**

None

**Notes**

This tag causes all enclosed text to be boldfaced.

<a name="cell"></a>
#### Cell

`[cell color="#EEEEEE" bgcolor="#111111" align="left" valign="top"][/cell]`

**Parameters**

*color*: OPTIONAL. Color of the text in the cells.

*bgcolor*: OPTIONAL. Background color of the cells.

*align*: OPTIONAL. Horizontal alignment of the cell contents. 

*valign*: OPTIONAL. Vertical alignment of the cell contents.

**Notes**

Defines a table cell. Only `cell` tags MUST be placed inside `row` or `header` tags. Table content MUST be placed inside `cell` tags. Non-compliant code MUST be ignored, and implementors are encouraged to remove non-compliant content.

<a name="code"></a>
#### Code / Preformatted

**Syntax**  

`[code language="markdown"]preformatted text[/code]`	

**Parameters**

*language*: OPTIONAL. Specifies a language, enabling syntax highlighting of the enclosed text.

**Notes**

This tag marks enclosed text as preformatted code. Such text MUST be rendered with a fixed-width font and whitespace MUST not be modified. A language MAY be specified, but syntax highlighting is not a requirement.

<a name="document"></a>
#### Document

**Syntax**  

```
[document language="en-us" color="#ffffff" image="tile.jpg" repeat="yes"]
.
.
.
[/document]
```

**Parameters**

*bgcolor*: OPTIONAL. Specifies the background color of the page. It may be specified as an RGB hextet, e.g. `#ff0033` or as a color name from the [Color Reference](#color-ref) section.

*color*: OPTIONAL. Specifies the default text color of the page. As with the `bgcolor` parameter, it may be specified as an RGB hextet, e.g. `#ff0033` or as a color name from the [Color Reference](#color-ref) section.

*height*: OPTIONAL. Specifies the height of the document in inches (in) or millimeters (mm). Note that some contexts, such as conversion to HTML may ignore this value. This parameter is also ignored if the `width` parameter is omitted.

*image*: OPTIONAL. Specifies the name of the image to be used as the document background.

*language*: OPTIONAL. Specifies the language in which the document is written, specified as an IETF language tag, e.g. "en-us".

*repeat*: OPTIONAL. Specifies whether or not the background image should repeat itself. Values MUST be *yes* or *no*. This parameter is ignored if the image does not exist or is invalid.

*width*: OPTIONAL. Specifies the width of the document in inches (in) or millimeters (mm). Note that some contexts, such as conversion to HTML may ignore this value. This parameter is also ignored if the `height` parameter is omitted.

**Notes**

This tag is used to enclose all text in the document. Any text preceding the opening tag or following the closing tag MUST be ignored or, even better, removed. Note that these parameters are all optional and MAY be ignored by the renderer.

<a name="header"></a>

#### Header

`[header color="#EEEEEE" bgcolor="#111111" align="left" valign="top"][/header]`

**Parameters**

*color*: OPTIONAL. Color of the text in the cells.

*bgcolor*: OPTIONAL. Background color of the cells.

*align*: OPTIONAL. Horizontal alignment of the cell contents. 

*valign*: OPTIONAL. Vertical alignment of the cell contents.

**Notes**

Defines a table header row. Only `cell` tags MUST be placed inside `row` or `header` tags. Table content MUST be placed inside `cell` tags. Non-compliant code MUST be ignored, and implementors are encouraged to remove non-compliant content.

<a name="image"></a>
#### Image

**Syntax**  

`[image name="image1.png" width="500" height="500"]Image Caption[/image]`  
`[image name="image2.png" width="50%" height="50%"]Image2 Caption[/image]`  

**Parameters**

*width*: OPTIONAL. Specifies the image's width in pixels or percent.

*height*: OPTIONAL. Specifies the image's height in pixels or percent

**Notes**

Unlike HTML, tag pairs are used with images in AnTM documents, with the enclosed text to be rendered as the image's caption. A number by itself MUST be considered a pixel size and a number ending in a percent symbol MUST be interpreted as a percentage relative to the image's size in pixels.

<a name="italic"></a>
#### Italics

`[i]Italicized text[/i]`	

**Parameters**

None

**Notes**

This tag marks enclosed text as italicized.

<a name="link"></a>
#### Link

`[link url="" name=""]Link title[/link]`	

**Parameters**

*url*: OPTIONAL. The address of the link.

*name*: OPTIONAL. The name of the link -- just like with named anchors in HTML.  

**Notes**

Text enclosed in this tag pair is a hyperlink. Links MUST be either a relative in-page link or a canonical remote link. As stated in the Embedded Content section, implementors MUST clearly display links as such and are strongly encouraged to add features to protect users from malicious links and enable them to quickly understand the destination of the link.  

<a name="list"></a>
#### List / List Item
```
[ulist style="disc"]
	[li]Item 1[/li]
	[li]Item 2[/li]
	[li]Item 3[/li]
[/ulist]
[olist style="upper-roman"]
	[li]Item A[/li]
	[li]Item B[/li]
	[li]Item C[/li]
[/olist]
```

**Parameters**

*style*: OPTIONAL. The style of list, such as `upper-roman`, `square`, or `lower-alpha`. For a complete list, consult the CSS3 list of styles for the property `list-style-type`.

The `li` tag has no parameters.  

**Notes**

These tags are for constructing unordered and ordered lists.

<a name="quote"></a>
#### Quote

`[quote]Quoted text[/quote]`	

**Parameters**

None

**Notes**

Displays the enclosed text in a block quote style.  

<a name="row"></a>
#### Row

`[row color="#EEEEEE" bgcolor="#111111" align="left" valign="top"][/row]`

**Parameters**

*color*: OPTIONAL. Color of the text in the cells.

*bgcolor*: OPTIONAL. Background color of the cells.

*align*: OPTIONAL. Horizontal alignment of the cell contents. 

*valign*: OPTIONAL. Vertical alignment of the cell contents.

**Notes**

Defines a table row. Only `cell` tags MUST be placed inside `row` or `header` tags. Table content MUST be placed inside `cell` tags. Non-compliant code MUST be ignored, and implementors are encouraged to remove non-compliant content.

<a name="strikeout"></a>
#### Strikeout

`[s]Strikeout text[/s]`	

**Parameters**

None

**Notes**

Displays the enclosed text with a single-line strikeout style.

<a name="style"></a>
#### Styled Text

`[style family="Arial" size="12" color="blue"]Styled text[/style]`	

**Parameters**

*Family*: OPTIONAL. The family name of the font to use.

*Size*: OPTIONAL. The size in points or, if *px* is appended to the size, pixels.  

*color*: OPTIONAL. The color of the text. This may be specified by RGB hextet or by color name. See the [Color Reference](#color-ref) section for details.

**Notes**

This tag pair provides a fast, simple, flexible way to adjust the look of text. Note that the font must already be installed on the user's system. Multiple font families MAY be specified in a comma-separated list as in HTML5 `font` tags. See the section [Standard Fonts](#standard-fonts) for more details on font availability.

<a name="subscript"></a>
#### Subscript

`[sub]subscripted text[/sub]`	

**Parameters**

None

**Notes**

Text enclosed by this tag pair is rendered as subscripted text.

<a name="superscript"></a>
#### Superscript

`[sup]superscripted text[/sup]`	

**Parameters**

None

**Notes**

Text enclosed by this tag pair is rendered as superscripted text.

<a name="table"></a>
#### Table / Row / Cell

`[table border="1" color="#EEEEEE" bgcolor="#111111" linecolor="#000000" align="left" valign="top"][/table]`

**Parameters**

*Border*: OPTIONAL. Thickness, in pixels, of the borders of the table.

*color*: OPTIONAL. Color of the text in the cells.

*bgcolor*: OPTIONAL. Background color of the cells.

*linecolor*: OPTIONAL. Color of the lines in between the cells.

*align*: OPTIONAL. Horizontal alignment of the cell contents. 

*valign*: OPTIONAL. Vertical alignment of the cell contents.

**Notes**

The `table` tags enclose table structure tags, i.e. `header` and `row` tags, and enable default styling of the rows and cells. Text and other content are expected to be placed within `cell` tags. Content not following the expected structure of a `table` construct, for example, a `cell` tag pair not inside a `header` or `row` tag pair, MUST be ignored or, better yet, removed.

<a name="underline"></a>
#### Underline

`[u]Underlined text[/u]`	

**Parameters**

None

**Notes**

Text enclosed by this tag pair is rendered as underlined text.


<a name="color-ref"></a>
### Color Reference

AnTM color references follow Web standards, supporting 140 different color names and associated hextet values.

<table>
<th>Color Name</th><th>Hex Value</th><th>Color</th>
<tr><td>AliceBlue</td><td>#F0F8FF</td><td style="background:#F0F8FF"></td></tr>
<tr><td>AntiqueWhite</td><td>#FAEBD7</td><td style="background:#FAEBD7"></td></tr>
<tr><td>Aqua</td><td>#00FFFF</td><td style="background:#00FFFF"></td></tr>
<tr><td>Aquamarine</td><td>#7FFFD4</td><td style="background:#7FFFD4"></td></tr>
<tr><td>Azure</td><td>#F0FFFF</td><td style="background:#F0FFFF"></td></tr>
<tr><td>Beige</td><td>#F5F5DC</td><td style="background:#F5F5DC"></td></tr>
<tr><td>Bisque</td><td>#FFE4C4</td><td style="background:#FFE4C4"></td></tr>
<tr><td>Black</td><td>#000000</td><td style="background:#000000"></td></tr>
<tr><td>BlanchedAlmond</td><td>#FFEBCD</td><td style="background:#FFEBCD"></td></tr>
<tr><td>Blue</td><td>#0000FF</td><td style="background:#0000FF"></td></tr>
<tr><td>BlueViolet</td><td>#8A2BE2</td><td style="background:#8A2BE2"></td></tr>
<tr><td>Brown</td><td>#A52A2A</td><td style="background:#A52A2A"></td></tr>
<tr><td>BurlyWood</td><td>#DEB887</td><td style="background:#DEB887"></td></tr>
<tr><td>CadetBlue</td><td>#5F9EA0</td><td style="background:#5F9EA0"></td></tr>
<tr><td>Chartreuse</td><td>#7FFF00</td><td style="background:#7FFF00"></td></tr>
<tr><td>Chocolate</td><td>#D2691E</td><td style="background:#D2691E"></td></tr>
<tr><td>Coral</td><td>#FF7F50</td><td style="background:#FF7F50"></td></tr>
<tr><td>CornflowerBlue</td><td>#6495ED</td><td style="background:#6495ED"></td></tr>
<tr><td>Cornsilk</td><td>#FFF8DC</td><td style="background:#FFF8DC"></td></tr>
<tr><td>Crimson</td><td>#DC143C</td><td style="background:#DC143C"></td></tr>
<tr><td>Cyan</td><td>#00FFFF</td><td style="background:#00FFFF"></td></tr>
<tr><td>DarkBlue</td><td>#00008B</td><td style="background:#00008B"></td></tr>
<tr><td>DarkCyan</td><td>#008B8B</td><td style="background:#008B8B"></td></tr>
<tr><td>DarkGoldenRod</td><td>#B8860B</td><td style="background:#B8860B"></td></tr>
<tr><td>DarkGray</td><td>#A9A9A9</td><td style="background:#A9A9A9"></td></tr>
<tr><td>DarkGrey</td><td>#A9A9A9</td><td style="background:#A9A9A9"></td></tr>
<tr><td>DarkGreen</td><td>#006400</td><td style="background:#006400"></td></tr>
<tr><td>DarkKhaki</td><td>#BDB76B</td><td style="background:#BDB76B"></td></tr>
<tr><td>DarkMagenta</td><td>#8B008B</td><td style="background:#8B008B"></td></tr>
<tr><td>DarkOliveGreen</td><td>#556B2F</td><td style="background:#556B2F"></td></tr>
<tr><td>DarkOrange</td><td>#FF8C00</td><td style="background:#FF8C00"></td></tr>
<tr><td>DarkOrchid</td><td>#9932CC</td><td style="background:#9932CC"></td></tr>
<tr><td>DarkRed</td><td>#8B0000</td><td style="background:#8B0000"></td></tr>
<tr><td>DarkSalmon</td><td>#E9967A</td><td style="background:#E9967A"></td></tr>
<tr><td>DarkSeaGreen</td><td>#8FBC8F</td><td style="background:#8FBC8F"></td></tr>
<tr><td>DarkSlateBlue</td><td>#483D8B</td><td style="background:#483D8B"></td></tr>
<tr><td>DarkSlateGray</td><td>#2F4F4F</td><td style="background:#2F4F4F"></td></tr>
<tr><td>DarkSlateGrey</td><td>#2F4F4F</td><td style="background:#2F4F4F"></td></tr>
<tr><td>DarkTurquoise</td><td>#00CED1</td><td style="background:#00CED1"></td></tr>
<tr><td>DarkViolet</td><td>#9400D3</td><td style="background:#9400D3"></td></tr>
<tr><td>DeepPink</td><td>#FF1493</td><td style="background:#FF1493"></td></tr>
<tr><td>DeepSkyBlue</td><td>#00BFFF</td><td style="background:#00BFFF"></td></tr>
<tr><td>DimGray</td><td>#696969</td><td style="background:#696969"></td></tr>
<tr><td>DimGrey</td><td>#696969</td><td style="background:#696969"></td></tr>
<tr><td>DodgerBlue</td><td>#1E90FF</td><td style="background:#1E90FF"></td></tr>
<tr><td>FireBrick</td><td>#B22222</td><td style="background:#B22222"></td></tr>
<tr><td>FloralWhite</td><td>#FFFAF0</td><td style="background:#FFFAF0"></td></tr>
<tr><td>ForestGreen</td><td>#228B22</td><td style="background:#228B22"></td></tr>
<tr><td>Fuchsia</td><td>#FF00FF</td><td style="background:#FF00FF"></td></tr>
<tr><td>Gainsboro</td><td>#DCDCDC</td><td style="background:#DCDCDC"></td></tr>
<tr><td>GhostWhite</td><td>#F8F8FF</td><td style="background:#F8F8FF"></td></tr>
<tr><td>Gold</td><td>#FFD700</td><td style="background:#FFD700"></td></tr>
<tr><td>GoldenRod</td><td>#DAA520</td><td style="background:#DAA520"></td></tr>
<tr><td>Gray</td><td>#808080</td><td style="background:#808080"></td></tr>
<tr><td>Grey</td><td>#808080</td><td style="background:#808080"></td></tr>
<tr><td>Green</td><td>#008000</td><td style="background:#008000"></td></tr>
<tr><td>GreenYellow</td><td>#ADFF2F</td><td style="background:#ADFF2F"></td></tr>
<tr><td>HoneyDew</td><td>#F0FFF0</td><td style="background:#F0FFF0"></td></tr>
<tr><td>HotPink</td><td>#FF69B4</td><td style="background:#FF69B4"></td></tr>
<tr><td>IndianRed</td><td>#CD5C5C</td><td style="background:#CD5C5C"></td></tr>
<tr><td>Indigo</td><td>#4B0082</td><td style="background:#4B0082"></td></tr>
<tr><td>Ivory</td><td>#FFFFF0</td><td style="background:#FFFFF0"></td></tr>
<tr><td>Khaki</td><td>#F0E68C</td><td style="background:#F0E68C"></td></tr>
<tr><td>Lavender</td><td>#E6E6FA</td><td style="background:#E6E6FA"></td></tr>
<tr><td>LavenderBlush</td><td>#FFF0F5</td><td style="background:#FFF0F5"></td></tr>
<tr><td>LawnGreen</td><td>#7CFC00</td><td style="background:#7CFC00"></td></tr>
<tr><td>LemonChiffon</td><td>#FFFACD</td><td style="background:#FFFACD"></td></tr>
<tr><td>LightBlue</td><td>#ADD8E6</td><td style="background:#ADD8E6"></td></tr>
<tr><td>LightCoral</td><td>#F08080</td><td style="background:#F08080"></td></tr>
<tr><td>LightCyan</td><td>#E0FFFF</td><td style="background:#E0FFFF"></td></tr>
<tr><td>LightGoldenRodYellow</td><td>#FAFAD2</td><td style="background:#FAFAD2"></td></tr>
<tr><td>LightGray</td><td>#D3D3D3</td><td style="background:#D3D3D3"></td></tr>
<tr><td>LightGrey</td><td>#D3D3D3</td><td style="background:#D3D3D3"></td></tr>
<tr><td>LightGreen</td><td>#90EE90</td><td style="background:#90EE90"></td></tr>
<tr><td>LightPink</td><td>#FFB6C1</td><td style="background:#FFB6C1"></td></tr>
<tr><td>LightSalmon</td><td>#FFA07A</td><td style="background:#FFA07A"></td></tr>
<tr><td>LightSeaGreen</td><td>#20B2AA</td><td style="background:#20B2AA"></td></tr>
<tr><td>LightSkyBlue</td><td>#87CEFA</td><td style="background:#87CEFA"></td></tr>
<tr><td>LightSlateGray</td><td>#778899</td><td style="background:#778899"></td></tr>
<tr><td>LightSlateGrey</td><td>#778899</td><td style="background:#778899"></td></tr>
<tr><td>LightSteelBlue</td><td>#B0C4DE</td><td style="background:#B0C4DE"></td></tr>
<tr><td>LightYellow</td><td>#FFFFE0</td><td style="background:#FFFFE0"></td></tr>
<tr><td>Lime</td><td>#00FF00</td><td style="background:#00FF00"></td></tr>
<tr><td>LimeGreen</td><td>#32CD32</td><td style="background:#32CD32"></td></tr>
<tr><td>Linen</td><td>#FAF0E6</td><td style="background:#FAF0E6"></td></tr>
<tr><td>Magenta</td><td>#FF00FF</td><td style="background:#FF00FF"></td></tr>
<tr><td>Maroon</td><td>#800000</td><td style="background:#800000"></td></tr>
<tr><td>MediumAquaMarine</td><td>#66CDAA</td><td style="background:#66CDAA"></td></tr>
<tr><td>MediumBlue</td><td>#0000CD</td><td style="background:#0000CD"></td></tr>
<tr><td>MediumOrchid</td><td>#BA55D3</td><td style="background:#BA55D3"></td></tr>
<tr><td>MediumPurple</td><td>#9370DB</td><td style="background:#9370DB"></td></tr>
<tr><td>MediumSeaGreen</td><td>#3CB371</td><td style="background:#3CB371"></td></tr>
<tr><td>MediumSlateBlue</td><td>#7B68EE</td><td style="background:#7B68EE"></td></tr>
<tr><td>MediumSpringGreen</td><td>#00FA9A</td><td style="background:#00FA9A"></td></tr>
<tr><td>MediumTurquoise</td><td>#48D1CC</td><td style="background:#48D1CC"></td></tr>
<tr><td>MediumVioletRed</td><td>#C71585</td><td style="background:#C71585"></td></tr>
<tr><td>MidnightBlue</td><td>#191970</td><td style="background:#191970"></td></tr>
<tr><td>MintCream</td><td>#F5FFFA</td><td style="background:#F5FFFA"></td></tr>
<tr><td>MistyRose</td><td>#FFE4E1</td><td style="background:#FFE4E1"></td></tr>
<tr><td>Moccasin</td><td>#FFE4B5</td><td style="background:#FFE4B5"></td></tr>
<tr><td>NavajoWhite</td><td>#FFDEAD</td><td style="background:#FFDEAD"></td></tr>
<tr><td>Navy</td><td>#000080</td><td style="background:#000080"></td></tr>
<tr><td>OldLace</td><td>#FDF5E6</td><td style="background:#FDF5E6"></td></tr>
<tr><td>Olive</td><td>#808000</td><td style="background:#808000"></td></tr>
<tr><td>OliveDrab</td><td>#6B8E23</td><td style="background:#6B8E23"></td></tr>
<tr><td>Orange</td><td>#FFA500</td><td style="background:#FFA500"></td></tr>
<tr><td>OrangeRed</td><td>#FF4500</td><td style="background:#FF4500"></td></tr>
<tr><td>Orchid</td><td>#DA70D6</td><td style="background:#DA70D6"></td></tr>
<tr><td>PaleGoldenRod</td><td>#EEE8AA</td><td style="background:#EEE8AA"></td></tr>
<tr><td>PaleGreen</td><td>#98FB98</td><td style="background:#98FB98"></td></tr>
<tr><td>PaleTurquoise</td><td>#AFEEEE</td><td style="background:#AFEEEE"></td></tr>
<tr><td>PaleVioletRed</td><td>#DB7093</td><td style="background:#DB7093"></td></tr>
<tr><td>PapayaWhip</td><td>#FFEFD5</td><td style="background:#FFEFD5"></td></tr>
<tr><td>PeachPuff</td><td>#FFDAB9</td><td style="background:#FFDAB9"></td></tr>
<tr><td>Peru</td><td>#CD853F</td><td style="background:#CD853F"></td></tr>
<tr><td>Pink</td><td>#FFC0CB</td><td style="background:#FFC0CB"></td></tr>
<tr><td>Plum</td><td>#DDA0DD</td><td style="background:#DDA0DD"></td></tr>
<tr><td>PowderBlue</td><td>#B0E0E6</td><td style="background:#B0E0E6"></td></tr>
<tr><td>Purple</td><td>#800080</td><td style="background:#800080"></td></tr>
<tr><td>RebeccaPurple</td><td>#663399</td><td style="background:#663399"></td></tr>
<tr><td>Red</td><td>#FF0000</td><td style="background:#FF0000"></td></tr>
<tr><td>RosyBrown</td><td>#BC8F8F</td><td style="background:#BC8F8F"></td></tr>
<tr><td>RoyalBlue</td><td>#4169E1</td><td style="background:#4169E1"></td></tr>
<tr><td>SaddleBrown</td><td>#8B4513</td><td style="background:#8B4513"></td></tr>
<tr><td>Salmon</td><td>#FA8072</td><td style="background:#FA8072"></td></tr>
<tr><td>SandyBrown</td><td>#F4A460</td><td style="background:#F4A460"></td></tr>
<tr><td>SeaGreen</td><td>#2E8B57</td><td style="background:#2E8B57"></td></tr>
<tr><td>SeaShell</td><td>#FFF5EE</td><td style="background:#FFF5EE"></td></tr>
<tr><td>Sienna</td><td>#A0522D</td><td style="background:#A0522D"></td></tr>
<tr><td>Silver</td><td>#C0C0C0</td><td style="background:#C0C0C0"></td></tr>
<tr><td>SkyBlue</td><td>#87CEEB</td><td style="background:#87CEEB"></td></tr>
<tr><td>SlateBlue</td><td>#6A5ACD</td><td style="background:#6A5ACD"></td></tr>
<tr><td>SlateGray</td><td>#708090</td><td style="background:#708090"></td></tr>
<tr><td>SlateGrey</td><td>#708090</td><td style="background:#708090"></td></tr>
<tr><td>Snow</td><td>#FFFAFA</td><td style="background:#FFFAFA"></td></tr>
<tr><td>SpringGreen</td><td>#00FF7F</td><td style="background:#00FF7F"></td></tr>
<tr><td>SteelBlue</td><td>#4682B4</td><td style="background:#4682B4"></td></tr>
<tr><td>Tan</td><td>#D2B48C</td><td style="background:#D2B48C"></td></tr>
<tr><td>Teal</td><td>#008080</td><td style="background:#008080"></td></tr>
<tr><td>Thistle</td><td>#D8BFD8</td><td style="background:#D8BFD8"></td></tr>
<tr><td>Tomato</td><td>#FF6347</td><td style="background:#FF6347"></td></tr>
<tr><td>Turquoise</td><td>#40E0D0</td><td style="background:#40E0D0"></td></tr>
<tr><td>Violet</td><td>#EE82EE</td><td style="background:#EE82EE"></td></tr>
<tr><td>Wheat</td><td>#F5DEB3</td><td style="background:#F5DEB3"></td></tr>
<tr><td>White</td><td>#FFFFFF</td><td style="background:#FFFFFF"></td></tr>
<tr><td>WhiteSmoke</td><td>#F5F5F5</td><td style="background:#F5F5F5"></td></tr>
<tr><td>Yellow</td><td>#FFFF00</td><td style="background:#FFFF00"></td></tr>
<tr><td>YellowGreen</td><td>#9ACD32</td><td style="background:#9ACD32"></td></tr>
</table>

<a name="standard-fonts"></a>
### Standard Fonts

As part of the pursuit of good design, some attention to typography is important. Although users may have preferences, having some standardized available typefaces for the platform would be prudent. These fonts MUST be available and are recommended, but not required, be the default fonts for client software.

- Noto Sans
- Noto Serif
- Noto Mono
- Fira Code

These font families have wide Unicode coverage and readability, enabling easy communication across many languages. Fira Code provides many helpful ligatures for code display.

### Final Notes

BBCode, which was the inspiration for AnTM, was born out of a need for safety and security for user-submitted forum posts combined with a desire for expressiveness. Conversion to other formats is expected. However, **UNDER NO CIRCUMSTANCES** should any programming language text be permitted to execute. **Ever.** Any code in the body of a document should be treated like text. For example, in converting AnTM to HTML, the first change to be made should be substituting `&lt;` and `&gt;` for < and > in order to prevent any HTML tags, particularly *&lt;script&gt;* tags, from being executed or rendered downstream. This is not to say that syntax highlighting is forbidden; any text contents outside of the AnTM tags are expected to be displayed, not executed or rendered. 

[![Build Status](https://travis-ci.org/neurobin/mdx_wikilink_plus.svg?branch=release)](https://travis-ci.org/neurobin/mdx_wikilink_plus)

Converts wikilinks (`[[wikilink]]`) to relative links, including support for [GitHub image variant](https://docs.github.com/en/free-pro-team@latest/github/building-a-strong-community/editing-wiki-content#linking-to-images-in-a-repository). Absolute links are kept as is (with an automatic label made from the file path part in the URL if label is not given explicitly).

**You must not use this extension with markdown.extensions.wikilinks. This extension is designed to provide the functionalities of markdown.extensions.wikilinks with some extra features. Choose either one.**

# Install

```bash
pip install mdx_wikilink_plus
```

# Wikilink syntax

The geneal formats are:

1. Without explicit label: `[[wikilink]]`
2. With explicit label: `[[ link | label ]]`
    - only supported for links not images
3. Image: `[[image.ext]]`
    - supports: .png, .jpg, .jpeg or .gif
4. Image alt text: `[[image.ext|alt=alternate text]]`

# Usage

`import markdown` then:

```python
text = "[[wikilink]]"
md = markdown.Markdown(extensions=['mdx_wikilink_plus'])
html = md.convert(text)
```

# Quick examples

`[[/path/to/file-name]]` will become:

```html
<p><a class="wikilink" href="/path/to/file-name">File Name</a></p>
```

`[[/path/to/file name.jpg| alt= alt text]]` will become:

```html
<p><img alt="alt text" class="wikilink-image" src="/path/to/file-name.jpg" /></p>
```

`[[https://www.example.com/example-tutorial]]` will become:

```html
<p><a class="wikilink" href="https://www.example.com/example-tutorial">Example Tutorial</a></p>
```

and `[[https://www.example.com/?a=b&b=c]]` will become:

```html
<p><a class="wikilink" href="https://www.example.com/?a=b&amp;b=c">www.example.com</a></p>
```


## Configuration

The configuration options are:

Config param | Default | Details
------------ | ------- | -------
base_url | `''` | Prepended to the file_path part of the URL. A `/` at the end of the base_url will be handled intelligently.
end_url | `''` | Appended to the file_path part of the URL. If end_url is given (non-empty), then any `/` at the end of the file_path part in the URL is removed. If the end_url matches the extension of the file_path part, it will be ignored, for example, if end_url is `.html` and the wikilink provided is `[[/path/to/myfile.html]]`, then the URL will be `/path/to/myfile.html` not `/path/to/myfile.html.html`.
url_whitespace | `'-'` | Replace all whitespace in the file_path path with this character (string) when building the URL.
url_case | `'none'` | Choose case in the file_path. Available options: lowercase, uppercase.
label_case | `'titlecase'` | Choose case of the label. Available options: titlecase, capitalize, none. Capitalize will capitalize the first character only.
html_class | `'wikilink'` | Set custom HTML classes on the anchor tag. It does not add classes rather it resets any previously set value.
image_class | `'wikilink-image'` | Set custom HTML classes on the anchor tag. It does not add classes rather it resets any previously set value.
build_url | `mdx_wikilink_plus.build_url` | A callable that returns the URL string. [Default build_url callable](#the-build_url-callable)

**None of the configs apply on absolute URLs except html_class and build_url. (Yes, label_case won't work either)**

### Configuration through meta data

Configuration can also be passed through metadata (markdown.extensions.meta).

We recognize the following template:

```md
wiki_base_url: /static/
wiki_end_url: 
wiki_url_whitespace: _
wiki_url_case: lowercase
wiki_label_case: capitalize
wiki_html_class: wikilink
wiki_image_class: wikilink-image

The first line of the document
```


### An example with configuration:


```python
md_configs = {
                'mdx_wikilink_plus': {
                    'base_url': '/static',
                    'end_url': '.html',
                    'url_case': 'lowercase',
                    'html_class': 'a-custom-class',
                    #'build_url': build_url, # A callable
                    # all of the above config params are optional
                },
             }


text = """
[[Page Name]]

[[/path/to/file-name.png|alt=demo image]]

[[/path/to/file name/?a=b&b=c]]
"""


md = markdown.Markdown(extensions=['mdx_wikilink_plus'], extension_configs=md_configs)
print(md.convert(text))
```

The output will be:

```html
<p><a class="a-custom-class" href="/static/page-name.html">Page Name</a></p>
<p><img alt="demo image" class="wikilink-image" src="/static/path/to/file-name.png" /></p>
<p><a class="a-custom-class" href="/static/path/to/file-name.html?a=b&amp;b=c">File Name</a></p>
```

!!! info
    `end_url` is added at the end of the file-path part in the URL.


# More examples

Our test markdown:

```md
[[wikilink]]    `[[wikilink]]`

[[/path/to/file name]]

[[/path/to/file_name]]

[[/path/to/file-name]]

[[/path/to/file name/?a=b&b=c]]

[[/path/to/file name.html]]

[[/path/to/file name.html?a=b&b=c]]

[[https://www.example.com/?]]

[[https://www.example.com/?a=b&b=c]]

[[https://www.example.com/example-tutorial]]

[[https://www.example.com/example-tutorial | Example Tutorial]]
```

## Output without any config

```html
<p><a class="wikilink" href="wikilink">Wikilink</a>    <code>[[wikilink]]</code></p>
<p><a class="wikilink" href="/path/to/file-name">File Name</a></p>
<p><a class="wikilink" href="/path/to/file_name">File Name</a></p>
<p><a class="wikilink" href="/path/to/file-name">File Name</a></p>
<p><a class="wikilink" href="/path/to/file-name/?a=b&amp;b=c">File Name</a></p>
<p><a class="wikilink" href="/path/to/file-name.html">File Name</a></p>
<p><a class="wikilink" href="/path/to/file-name.html?a=b&amp;b=c">File Name</a></p>
<p><a class="wikilink" href="https://www.example.com/">www.example.com</a></p>
<p><a class="wikilink" href="https://www.example.com/?a=b&amp;b=c">www.example.com</a></p>
<p><a class="wikilink" href="https://www.example.com/example-tutorial">Example Tutorial</a></p>
<p><a class="wikilink" href="https://www.example.com/example-tutorial">Example Tutorial</a></p>
```

## With a config

With the configuration

```python
'mdx_wikilink_plus': {
    'base_url': '/static',
    'end_url': '.html',
    'url_whitespace': '-',
    'label_case': 'titlecase',
    'html_class': 'a-custom-class',
},
```

the output will be:

```html
<p><a class="a-custom-class" href="/static/wikilink.html">Wikilink</a>    <code>[[wikilink]]</code></p>
<p><a class="a-custom-class" href="/static/path/to/file-name.html">File Name</a></p>
<p><a class="a-custom-class" href="/static/path/to/file_name.html">File Name</a></p>
<p><a class="a-custom-class" href="/static/path/to/file-name.html">File Name</a></p>
<p><a class="a-custom-class" href="/static/path/to/file-name.html?a=b&amp;b=c">File Name</a></p>
<p><a class="a-custom-class" href="/static/path/to/file-name.html">File Name</a></p>
<p><a class="a-custom-class" href="/static/path/to/file-name.html?a=b&amp;b=c">File Name</a></p>
<p><a class="a-custom-class" href="https://www.example.com/">www.example.com</a></p>
<p><a class="a-custom-class" href="https://www.example.com/?a=b&amp;b=c">www.example.com</a></p>
<p><a class="a-custom-class" href="https://www.example.com/example-tutorial">Example Tutorial</a></p>
<p><a class="a-custom-class" href="https://www.example.com/example-tutorial">Example Tutorial</a></p>
```

## With meta (`markdown.extensions.meta`)

With the following meta added to the markdown:

```md
wiki_base_url: /static/
wiki_end_url: 
wiki_url_whitespace: _
wiki_label_case: capitalize
wiki_html_class: wikilink
```

the output will be:

```html
<p><a class="wikilink" href="/static/wikilink">Wikilink</a>    <code>[[wikilink]]</code></p>
<p><a class="wikilink" href="/static/path/to/file_name">File name</a></p>
<p><a class="wikilink" href="/static/path/to/file_name">File name</a></p>
<p><a class="wikilink" href="/static/path/to/file-name">File name</a></p>
<p><a class="wikilink" href="/static/path/to/file_name/?a=b&amp;b=c">File name</a></p>
<p><a class="wikilink" href="/static/path/to/file_name.html">File name</a></p>
<p><a class="wikilink" href="/static/path/to/file_name.html?a=b&amp;b=c">File name</a></p>
<p><a class="wikilink" href="https://www.example.com/">www.example.com</a></p>
<p><a class="wikilink" href="https://www.example.com/?a=b&amp;b=c">www.example.com</a></p>
<p><a class="wikilink" href="https://www.example.com/example-tutorial">Example tutorial</a></p>
<p><a class="wikilink" href="https://www.example.com/example-tutorial">Example Tutorial</a></p>
```


# The build_url callable

You can view the default [build_url](https://github.com/neurobin/mdx_wikilink_plus/blob/master/mdx_wikilink_plus/mdx_wikilink_plus.py#L36) function [here](https://github.com/neurobin/mdx_wikilink_plus/blob/master/mdx_wikilink_plus/mdx_wikilink_plus.py#L36).


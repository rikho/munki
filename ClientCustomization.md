# Introduction #

Munki 2's new Managed Software Center can be (and probably should be!) customized for your organization. Further, you can customize it per site, workgroup, or even user. Customization requires some familiarity with basic web development tools and techniques.

# Details #

Most of what Managed Software Center displays is within a WebKit WebView. The views are generated dynamically, combining HTML templates with live data and static resources (images, CSS, JavaScript).

The default templates are in `Managed Software Center.app/Contents/Resources/templates/` and the static assets are in `Managed Software Center.app/Contents/Resources/WebResources/`

One approach to customization would be to replace some of these files and (optionally) add additional files to these directories. You'd have to be certain to re-do your modifications each time Managed Software Center.app was updated, so that might be a tedious method of customization.

Instead, you can create one or more zip archives of your desired customization files and make them available at a URL, typically in your Munki repo. `managedsoftwareupdate` will attempt to download any client customization resources when it runs; Managed Software Center.app will use any custom resources downloaded by `managedsoftwareupdate`.

## Customization overview ##

While you _can_ customize anything in `Managed Software Center.app/Contents/Resources/templates/`, the most common customizations (and the only ones officially supported) are for these template files:

  * `showcase_template.html` -- controls the banner images and any links
  * `sidebar_template.html` -- the right-side sidebar displayed in the main Software view
  * `footer_template.html` -- the page footer

While it is possible to customize any of the other template files, it's possible (or even likely) that a future release of Managed Software Center.app will include changes to the default/included versions of these template files. This could lead to unexpected/undesired behavior if you did not also update your versions of the customized files.

### Showcase banner images requirements ###

When customizing the showcase\_template.html, you will typically want to provide your own banner images.

Banner images may be any format that WebKit can display natively; png might be a good choice.
Images should be 1158x200 (or at least look good when resized to that resolution). Since the main window can be resized, you should test the images to make sure they don't hide any important information when the window is sized smaller than 1158 pixels wide. (The window can be as small as 1000px wide, so the displayed part of the image will be even smaller than that: it's approximately 960px wide on Mavericks, but exact measurements might vary on different OS versions.)

### showcase\_template.html example ###

Here's an example of a customized showcase\_template.html. Note that this is a copy of the file at `Managed Software Center.app/Contents/Resources/templates/showcase_template.html`. **The JavaScript at the beginning of the file is left unchanged** (though you could change it if you wanted different behaviors).  In this example, we are only customizing the actual images shown and what they link to.

```
<script type="text/javascript">
var currentSlide = 0, playing = 1

function slides(){
    return document.querySelectorAll('div.stage>img')
}

function showSlide(slideNumber){
    theSlides = slides()
    for (c=0; c<theSlides.length; c++) {
        theSlides[c].style.opacity="0";
    }
    theSlides[slideNumber].style.opacity="1";
}

function showNextSlide(){
    if (playing) {
        currentSlide = (currentSlide > slides().length-2) ? 0 : currentSlide + 1;
        showSlide(currentSlide);
    }
}

function stageClicked() {
    slide = slides()[currentSlide];
    target = slide.getAttribute('target');
    if (target == '_blank') {
        window.AppController.openExternalLink_(slide.getAttribute('href'));
    } else {
        window.location.href = slide.getAttribute('href');
    }
}

window.onload=function(){
    showSlide(0);
    if (slides().length > 1) {
        setInterval(showNextSlide, 7500);
    }
}
</script>

<div class="showcase">
    <div class="stage" onClick='stageClicked();'>
        <img target="_blank" href="http://www.apple.com" alt="Apple" src="custom/resources/Apple.png" />
        <img href="detail-GoogleChrome.html"  alt="Google Chrome" src="custom/resources/Chrome.png" />
        <img href="developer-Google.html" alt="Google Applications" src="custom/resources/Google.png" />
    </div>
</div>
```

We have three images, `"custom/resources/Apple.png"`, `"custom/resources/Chrome.png"`, and `"custom/resources/Google.png"`. These will be in our archive of custom files. You could, if you wished, link to a full (external) http URL for these images, but Managed Software Center.app would display broken image link placeholders if it can't reach those URLs (for example, if the images are hosted on an internal web server not available when a user is at home or out of the office).

There are two optional attributes that can be added to each `img` element: `target` and `href`. `href` is a link -- it can point to any full or relative URL. if `target="_blank"`, the URL will be forwarded to the user's default browser; otherwise Managed Software Center.app will attempt to display the contents of the URL.

Notice the special links for the second and third images. These are URLs internal to Managed Software Center.app to cause it to display specific content. `"detail-GoogleChrome.html"` links to the detail view for GoogleChrome; `"developer-Google.html"` shows all items whose developer is "Google". More documentation on these internal links will be available in the future.

### sidebar\_template.html example ###

Here's an example of a customized sidebar\_template.html showing some of the possibilities:

```
<div class="sidebar">
    <div class="chart titled-box quick-links">
        <h2>Quick Links</h2>
        <div class="content">
            <ol class="list">
                <li class="link user-link"><a href="#">Welcome</a></li>
                <li class="link user-link"><a href="#">Support</a></li>
                <li class="separator"><hr/></li>
                <li class="popup">
                    <div class="select links">
                        <label>
                            <span></span>
                            <select id="category-selector" onchange="category_select()">
                                ${category_items}
                            </select>
                        </label>
                    </div>
                </li>
                <li class="link"><a href="http://www.apple.com/osx/whats-new/">What's new in Mavericks</a></li>
                <li class="link"><a target="_blank" href="http://www.apple.com">Apple</a></li>
                <li class="link"><a target="_blank" href="http://google.com">Search Google</a></li>
                <li class="link"><a target="_blank" href="http://bing.com">Search Bing</a></li>
                <li class="separator"><hr/></li>
                <li class="link"><a target="_blank" href="http://www.apple.com/support/">Apple support</a></li>
                <li class="link"><a target="_blank" href="http://www.apple.com/support/mac/">Mac</a></li>
                <li class="link"><a target="_blank" href="http://www.apple.com/support/osx/">OS X</a></li>
                <li class="link"><a target="_blank" href="http://www.apple.com/support/mac-apps/">Mac Apps</a></li>
            </ol>
        </div>
    </div>
</div>
```

The customized sidebar would look like this:

![http://wiki.munki.googlecode.com/git/images/customized_sidebar.png](http://wiki.munki.googlecode.com/git/images/customized_sidebar.png)

### footer\_template.html example ###

Finally, an example of a customized footer\_template.html:

```
<div class="bottom-links">
    <ul class="list" role="presentation">
        <li><a target="_blank" href="http://www.apple.com">Apple</a></li>
        <li><a target="_blank" href="http://www.google.com">Google</a></li>
        <li><a href="updates.html">Updates</a></li>
    </ul>
</div>
```

## Creating the client customization archive ##

The archive of client customization files has a specific format.
Create two directories: "templates" and "resources". Place all your customized template files in "templates"; custom images, css and JavaScript (or anything else referred to in your customized templates) should go in "resources".

I recommend that you use the command-line `zip` tool to create the actual archive. Using the Finder's "Compress foo" command will not create archives with the expected layout. Assuming you are in the directory containing your templates and resources directories:

```
zip -r site_default.zip resources/ templates/
```

`-r` tells zip to add the resources and templates directories recursively to the archive named site\_default.zip (without the '-r' flag, you'll get only empty directories in the archive!).

To verify the contents of an archive you've created, use
```
unzip -l site_default.zip
```
You'll see a listing of the archive contents.

A sample archive is here: http://downloads.munki.googlecode.com/git/site_default.zip

## Making client customization archives available to clients ##

`managedsoftwareupdate` as part of its update check, attempts to download client customization resources. Typically, you can make these available from your Munki repo by creating a "client\_resources" directory at the top level of the repo. If you want to make these available at a different URL, you can set Munki's **ClientResourceURL** to an alternate base URL. (This follows the pattern of ManifestURL, CatalogURL and PackageURL as alternate base URLs.)

If Munki's **ClientResourcesFilename** preference is defined, this filename will be used (appending ".zip") if needed; otherwise `managedsoftwareupdate` will request an archive with the same name as the primary manifest (plus ".zip"), falling back to "site\_default.zip".

Requests will be of the form "http://munki/repo/client_resources/site_default.zip". If no client customization resources are found, the default resources within Managed Software Update.app will be used.

## Behind the scenes ##

Client customization resources are downloaded to Munki's data directory; typically `/Library/Managed Installs`, in a `client_resources` subdirectory, with the filename `custom.zip` -- no matter the name of the remote file at the URL.

On launch, Managed Software Center.app checks for the existence of this file. If it exists, it's checked to see if it has the expected layout (top-level "resources" and "templates" directories), and if so, these directories are expanded into `~/Library/Caches/com.googlecode.munki.ManagedSoftwareCenter/html`, which is where Managed Software Center.app finds all of its HTML/CSS/JavaScript, etc resources.

You can open these same files (the ones in `~/Library/Caches/com.googlecode.munki.ManagedSoftwareCenter/html`) in Safari (and use Safari's web inspector, etc) to debug if things aren't behaving as you'd expect.
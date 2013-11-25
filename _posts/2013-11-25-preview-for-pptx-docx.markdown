---
layout: post
title: Preview for pptx, docx, pdf using google docs viewer
date: 2013-11-25 18:12
categories: episerver google
---
Hello

<h4>Models/Media/Document.cs</h4>
{% highlight c# %}
    using EPiServer.Core;
    using EPiServer.DataAnnotations;
    using EPiServer.Framework.DataAnnotations;
    using EPiServer.Shell;

    namespace EPiServer.Templates.Alloy.Models.Media
    {
        [ContentType(GUID = "E1D862F6-9E8D-4F0C-822F-028FD7FAD33E")]
        [MediaDescriptor(ExtensionString = "docx,pdf,txt,pptx,xlsx")]
        public class Document : MediaData 
        {
            /// <summary>
            /// Gets or sets the copyright for the document.
            /// </summary>
            public virtual string Copyright { get; set; }
        }

        [UIDescriptorRegistration]
        public class DocumentUIDescriptor : UIDescriptor<Document>
        {
            public DocumentUIDescriptor()
            {
                DefaultView = CmsViewNames.OnPageEditView;
            }
        }
    }
{% endhighlight %}

<h4>Controllers/DocumentController.cs</h4>
{% highlight c# %}
    using System;
    using System.Web.Mvc;
    using EPiServer.Framework.DataAnnotations;
    using EPiServer.Framework.Web;
    using EPiServer.Templates.Alloy.Models.Media;
    using EPiServer.Web;
    using EPiServer.Web.Mvc;
    using EPiServer.Web.Routing;

    namespace EPiServer.Templates.Alloy.Controllers
    {
        [TemplateDescriptor(
            Default = true,
            Inherited = true,
            AvailableWithoutTag = false,
            TemplateTypeCategory = TemplateTypeCategories.MvcController,
            Tags = new[] { RenderingTags.Edit, RenderingTags.Preview })]
        public class DocumentController : PartialContentController<Document>
        {
            private readonly UrlResolver _urlResolver;
            private readonly SiteDefinition _siteDefinition;

            public DocumentController(UrlResolver urlResolver, SiteDefinition siteDefinition)
            {
                _urlResolver = urlResolver;
                _siteDefinition = siteDefinition;
            }

            /// <summary>
            /// The index action for the image file. Creates the view model and renders the view.
            /// </summary>
            /// <param name="currentContent">The current image file.</param>
            public override ActionResult Index(Document currentContent)
            {  
                //Generate external url
                var uri = new Uri(_siteDefinition.SiteUrl, _urlResolver.GetUrl(currentContent, new VirtualPathArguments
                {
                    ContextMode = ContextMode.Default
                }));
                
                return View(new DocumentModel
                {
                    Url = uri.ToString(),
                    Name = currentContent.Name
                });
            }
        }

        public class DocumentModel
        {
            public string Url { get; set; }
            public string Name { get; set; }
        }
    }
{% endhighlight %}

<h4>Views/Document/Index.cshtml</h4>
{% highlight html %}
    @model EPiServer.Templates.Alloy.Controllers.DocumentModel
    @{
        Layout = null;
    }

    <!DOCTYPE html>

    <html>
    <head>
        <meta name="viewport" content="width=device-width" />
        <title>Document</title>
        <style>
            html {
                height: 100%;
            }
            body {
                margin: 0;
                width: 100%;
                height: 100%;
            }
            iframe {
                width: 100%;
                height: 100%;
            }
        </style>
    </head>
    <body>
            <iframe src="http://docs.google.com/viewer?url=@Model.Url&&embedded=true"></iframe>
    </body>
    </html>
{% endhighlight %}
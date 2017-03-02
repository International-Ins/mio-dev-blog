---
layout: post
title: Dumb Templates, Smart Design
excerpt_separator: <!-- /excerpt -->
---

What good is having code running inside your templates, anyway?

<!-- /excerpt -->

The [MVC pattern](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) is a wonderful thing.  It helps developers design resilient, testable, and well structured applications.  Almost every web framework is centered around MVC ([Phoenix](http://www.phoenixframework.org/), withstanding).  So why, then, do most of these frameworks allow you to embed code directly into the view layer?

View layers tend to allow code because developers immediately run into problems when they need to iterate over a list of items.  And, let's face it, with a CRUD operation, that's one of the first things you do.  One can justify leaving code in the view layer, because it adds flexibility.  The last thing you want is to have your view templates be a limiting factor in your application.

Mexico Insurance Online's application, however, is a solid exception to that rule.  We have some... *substantial* resellers who sell directly to their customers through the application.  Those resellers demand an entirely custom view.  From the font on a &lt;label&gt; tag to which fields appear on a given form, everything can, and will, be customized.  To that end, a bit of rigidity actually makes life easier, let me show you.

[Curly Templates](https://github.com/zendesk/curly) is what Mexico Insurance Online's application will use for the view layer in our rebuild project.  The magic of Curly lies in its Presenter classes:

```ruby
class Posts::CommentPresenter < Curly::Presenter
  presents :comment
  def replies
    content_tag :div do
      for reply in @comment.replies do
        render 'reply', reply
      end
    end
  end
end
```

A single presenter, such as the above, can be shared across multiple views.  If a resellers needs to customize the way that replies are presented, then an alternative to the `_reply.html.curly` file can be created, leaving the rest of the view un-touched.

Had this view been implemented with the ERB syntax that comes with Ruby on Rails, then both the main application and the custom view for the reseller would need the loop code.  Given that Mexico Insurance Online has 296 white-label templates at the moment, this means there would be 296 loops to update if something changed.

This may seem like something simple, but the white-labeling provided by Mexico Insurance Online is crucial to its continued success.
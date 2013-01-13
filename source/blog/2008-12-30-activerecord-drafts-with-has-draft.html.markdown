---
title: "ActiveRecord Drafts with has_draft"
tags: Rails Plugins, Rails Techniques
---

I ran into a problem a while back of creating draft copies of ActiveRecord models for the purpose of establishing a draft/live system. I've since found a reason to resurrect this and publish it to GitHub and clean some things up. Check out has_draft on GitHub.

has_draft allows for multiple "drafts" of a model which can be useful when developing:
* Draft/Live Version of Pages, for examples
* A workflow system whereby a live copy may need to be active while a draft copy is awaiting approval.

The semantics of this as well as most of the inspiration comes from version_fu, an excellent plugin for a similar purpose of maintaining several "versions" of a model.

This was built to be able to be tacked on to existing models, so the data schema doesn't need to change at all for the model this is applied to. As such, drafts are actually stored in a nearly-identical table and there is a has_one relationship to this. This separation allows the base model to really be treated just as before without having to apply conditions in queries to make sure you are really getting the "live" (non-draft) copy: Page.all will still only return the non-draft pages. This separate table is backed by a model created on the fly as a constant on the original model class. For example if a Page has_draft, a Page::Draft class will exist as the model for the page_drafts table.

## Basic Example ##

    ## First Migration (If Creating base model and drafts at the same time):
    class InitialSchema < ActiveRecord::Migration

      [:articles, :article_drafts].each do |table_name|
        create_table table_name, :force => true do |t|
          t.references :article if table_name == :article_drafts

          t.string :title
          t.text :summary
          t.text :body
          t.date :post_date
        end
      end
    end

    ## Model Class
    class Article < ActiveRecord::Base
      has_draft
    end

    ## Exposed Class Methods & Scopes:
    Article.draft_class
    => Article::Draft
    Article.with_draft.all
    => (Articles that have an associated draft)
    Article.without_draft.all
    => (Articles with no associated draft)

    ## Usage Examples:
    article = Article.create(
      :title => "My Title",
      :summary => "Information here.",
      :body => "Full body",
      :post_date => Date.today
    )

    article.has_draft?
    => false

    article.instantiate_draft!

    article.has_draft?
    => true

    article.draft
    => Article::Draft Instance

    article.draft.update_attributes(
      :title => "New Title"
    )

    article.replace_with_draft!

    article.title
    => "New Title"

    article.destroy_draft!

    article.has_draft?
    => false

## Custom Options ##

    ## First Migration (If Creating base model and drafts at the same time):
    class InitialSchema < ActiveRecord::Migration

      [:articles, :article_copies].each do |table_name|
        create_table table_name, :force => true do |t|
          t.integer :news_article_id if table_name == :article_copies

          t.string :title
          t.text :summary
          t.text :body
          t.date :post_date
        end
      end

    end

    ## Model Class
    class Article < ActiveRecord::Base
      has_draft :class_name => 'Copy', :foreign_key => :news_article_id, :table_name => 'article_copies'
    end

## Method Callbacks: ##

There are three callbacks you can specify directly as methods.

    class Article < ActiveRecord::Base
      has_draft

      def before_instantiate_draft
        # Do Something
      end

      def before_replace_with_draft
        # Do Something
      end

      def before_destroy_draft
        # Do Something
      end
    end

## Block of Code Run for Draft Class: ##

Because you don't directly define the draft class, you can specify a block of code to be run in its
context by passing a block to has_draft.

    class Article < ActiveRecord::Base
      belongs_to :user

      has_draft do
        belongs_to :last_updated_user

        def approve!
          self.approved_at = Time.now
          self.save
        end
      end

    end
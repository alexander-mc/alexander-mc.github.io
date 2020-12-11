---
layout: post
title:      "Having attachment issues? Paperclip vs. CarrierWave vs. Active Storage"
date:       2020-12-11 06:17:48 +0000
permalink:  having_attachment_issues_paperclip_vs_carrierwave_vs_active_storage
---


Looking for a quick and easy way to allow users to upload a file in your Rails app? Among the most popular solutions are Active Storage and the Paperclip and CarrierWave gems. But which is best?

If you are developing a simple app, then go with CarrierWave. As someone who tries to avoid over-gemifying a project, I do not regret my decision to use CarrierWave in creating my last product.

### Rationale

At first, I attempted to use [Active Storage](https://guides.rubyonrails.org/active_storage_overview.html). However, after spending a few minutes reading the Rails guide documentation, I realized I needed something simpler. Per the documentation, “Active Storage facilitates uploading files to a cloud storage service like Amazon S3, Google Cloud Storage, or Microsoft Azure Storage and attaching those files to Active Record objects.” My product is in the very early stages of development and runs on a local server, and so trying to read around the language about cloud storage services was a little frustrating.

I also considered using the Paperclip gem. After all, on [The Ruby Toolbox](https://www.ruby-toolbox.com) Paperclip has more downloads than CarrierWave (46.6M vs 38.6M). It was also released a year earlier, so I was attracted to its appeal as “the Original” file uploader gem (compared to CarrierWave, at least). However, Paperclip’s latest release was in 2018 whereas CarrierWave released a new version this year. Even more, Paperclip has been deprecated! This is mentioned on Ruby Toolbox and in Paperclip's formal announcement [here](https://thoughtbot.com/blog/closing-the-trombone).

So after axing Active Storage and Paperclip, I finally decided to try CarrierWave – and I’m glad I did.

### How it works

**1. Install the gem.** Put `gem ‘carrierwave’` into your gemfile and then run `bundle install` in your console

**2. Set up the model.** Create a model with a column titled “attachment” and a type of “string. If you already have a model set up, then you will need to create a migration to add a column, like this:

```
class AddFilePathToReports < ActiveRecord::Migration[6.0]
  def change
    add_column :reports, :attachment, :string
  end
end
```

Afterwards, be sure to run `rake db:migrate` in your console.

**3. Add the uploader.** Run the following command: `rails g uploader attachment`

**4. Mount the uploader.** Add a couple lines in your model to mount the uploader and validate, like so:

```
class Report < ApplicationRecord
    mount_uploader :attachment, AttachmentUploader
    validates :attachment, presence: { message: "was not entered" }
end
```

**5. Permit the attachment.** In your controller, allow the attachment in your params. Your controller should look something like this,

```
class ReportsController < ApplicationController

   private

   def report_params
        params.require(:report).permit(:report_name, :attachment)
   end

end
```

**6. Add an attachment field in your form.** Go to your view and add an attachment field in the form, like this:

```
<%= form_for @report, url: reports_path(report) do |f| %>
   <%= f.label :report_name %>
   <%= f.text_field :report_name %>
   <%= f.label :attachment %>
   <%= f.file_field :attachment %>
   <%= f.submit "Create Report" %>
<% end %>
```

**7. Other options.** By now, in your ‘app’ directory, you should see a folder titled ‘uploaders’ with a file called ‘attachment_uploader.rb’. Check it out! You will notice that you can edit your uploader to only accept certain file types. All you need to do is uncomment and edit a few lines of code, like so:

```
def extension_whitelist
  %w(json)
end
```

That’s it!

Want some practice? Check out this amazing tutorial: https://www.tutorialspoint.com/ruby-on-rails/rails-file-uploading.htm 

#### Resources:
* [TutorialsPoint](https://www.tutorialspoint.com/ruby-on-rails/rails-file-uploading.htm)
* [CarrierWave](https://github.com/carrierwaveuploader/carrierwave)
* [Pluralsight](https://www.pluralsight.com/guides/handling-file-upload-using-ruby-on-rails-5-api)


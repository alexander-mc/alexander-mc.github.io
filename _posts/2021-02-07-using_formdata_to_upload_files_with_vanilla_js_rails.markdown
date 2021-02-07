---
layout: post
title:      "Using FormData to upload files with vanilla JS + Rails"
date:       2021-02-07 16:35:19 -0500
permalink:  using_formdata_to_upload_files_with_vanilla_js_rails
---


After spending several hours researching how to upload MIDI data (stored as a blob) so that it can be saved into my Google Cloud Storage (GCS) backend, I realized that processing uploads is far less facile than I initially conceived. This post details two takeaways about file uploading that I learned from developing my most recent product. It is geared towards beginning JavaScript and Rails developers who are familiar with how to process uploads in Rails (such as through Active Storage/CarrierWave/Paperclip) but less familiar with how to process them in the frontend using vanilla JS.

### TL;DR
***

**Takeaway #1:** Assuming one’s strong params follow the popular convention of params.require(:mode_name).permit(:attr_name_1 …  attr_name_Z), each key in the FormData object used to post the blob or file must reference the model name and permitted attribute as ‘model_name[attr_name]’.


**Takeaway #2:** When sending a FormData object with the Fetch API, do not include a content-type header.


Don’t worry if these takeaways don’t make much sense now. I promise, they will by the end of the post. Let’s get started…

### The backend (Rails)
***

First, in your controller, make sure you have your strong params set up to accept the file being uploaded. In my case, I needed to upload a blob with MIDI data. Your params should look something like this:

```
    private

    def song_params
        params.require(:song).permit(:title, :attachment)
    end
```

Then, I used [Active Storage](https://edgeguides.rubyonrails.org/active_storage_overview.html) to upload the file into Google Cloud Storage.


#### A side note

Three of the main ways to upload files on the server-side with Rails is through Active Storage, CarrierWave, or Paperclip. I’ve tried each one and wrote a brief post comparing them [here](https://alexander-mc.github.io/havingattachmentissuespaperclipvscarrierwavevsactivestorage). If you are a beginning developer and are unsure which to use, I recommend using Active Storage if

1. You are developing an app where you might like to store data in a cloud service such as Amazon S3, Google Cloud Storage, or Microsoft Azure Storage
2. You have time to go through the [documentation](https://edgeguides.rubyonrails.org/activestorageoverview.html) (like all Edge Guides, it’s really good)
 
However, if you're very new to programming and going through the guide seems too intimidating right now (it was for me, at first!), or if you just need a quick-and-dirty way to upload a file, then CarrierWave will work great. As for Paperclip, it’s deprecated, so don’t bother. Oh, and one last piece of advice, Active Storage will work even if all you want to do is  upload files onto a local disk (i.e. you don’t have to connect to a cloud service to use it).

### The frontend (vanilla JS)
***

Setting up the frontend to process uploads is where things can get tricky. Essentially, you need to:

1. Instantiate a [FormData object]( https://developer.mozilla.org/en-US/docs/Web/API/FormData/FormData)
2. Store your blob or file into the FormData object
3. Send a POST request to your backend, using the FormData object as the body of your request

Sounds easy, right?

#### The wrong approach

Let’s say we followed the above outline and ended up with something like this…

```
function saveMIDI (songTitle, songBlob) {

    const url = ‘my/url/path’;
    const formData = new FormData();

    formData.append(“title”, songTitle);
    formData.append(“attachment”, songBlob);

    fetch (url, {
        method: 'POST',
        body: formData
    })
    .then(resp => resp.json())
    .then(json => {
	
	// SOME CODE

    })
    .catch(error => {

	// SOME MORE CODE        

    });
}
```

We created a FormData object, appended some data to it, and then used it as the body of our POST request. Looks good!

Unfortunately, our file won’t be uploaded – what gives?

#### The correct approach

Let’s look inside our console. Using the above example would generate something like this:

```
=> <ActionController::Parameters {"title"=>"myAmazingSong", "attachment"=>#<ActionDispatch::Http::UploadedFile:0x00007fb8b8bce1c0 @tempfile=#<Tempfile:/var/folders/e1/5yw3_l2gn/T/RackMultipart20210112-33716-1n7t60u.midi> … permitted: false>
```


Remember those strong params? We’ve required “song”, but it isn’t mentioned anywhere in our console’s response! In fact, what we want is something like this (notice how the “title” and “attachment” attributes are nested within “song”).

```
=> <ActionController::Parameters {"song"=>{"title"=>" myAmazingSong ", "attachment"=>#< ActionDispatch::Http::UploadedFile:0x00007fb8b8bce1c0 @tempfile=#<Tempfile:/var/folders/e1/5yw3_l2gn/T/RackMultipart20210112-33716-1n7t60u.midi> … permitted: false>
```

So, our function should actually look more like the following…

```
function saveMIDI (songTitle, songBlob) {

    const url = ‘my/url/path’;
    const formData = new FormData();

    formData.append(‘song[title]', songTitle);
    formData.append(‘song[attachment]', songBlob);

    fetch (url, {
        method: 'POST',
        body: formData
    })
    .then(resp => resp.json())
    .then(json => {
	
	// SOME CODE

    })
    .catch(error => {

	// SOME MORE CODE        

    });
}


```

Now, the ‘title’ and ‘attachment’ attributes will be nested under ‘song’, corresponding with the format of our strong params, which was set up when we required ‘song’ and permitted ‘title’ and ‘attachment’ in the song_params method of our controller.

**Takeaway #1: Assuming one’s strong params follow the popular convention of params.require(:mode_namel).permit(:attr_name_1 …  attr_name_Z), each key in the FormData object used to post the blob or file must reference the model name and permitted attribute as ‘model_name[attr_name]’.**

Also, take a look at the fetch request. Notice that we did not include a ‘Content-Type’ header. This was intentional and brings me to my second takeaway:

**Takeaway #2: When sending a FormData object with the Fetch API, do not include a content-type header.**


### Conclusion
***
Processing files using a JavaScript frontend and Rails backend can be difficult. Two incredibly important nuances about file uploading that I learned while creating my last app are to ensure that the FormData object follows the hash structure of the strong params in the controller and to exclude the content-type header when posting the fetch request with FormData. Hopefully, these lessons and the above examples will help new JS/Rails developers learning to process a file upload with FormData for their first time. Best of luck!


### Resources
***
* [Uploading Files To Your Rails API]( https://itnext.io/uploading-files-to-your-rails-api-6b293a4a5c90)
* [Having attachment issues? Paperclip vs. CarrierWave vs. Active Storage]( https://alexander-mc.github.io/having_attachment_issues_paperclip_vs_carrierwave_vs_active_storage)
* [Active Storage](https://edgeguides.rubyonrails.org/active_storage_overview.html)
* [FormData()]( https://developer.mozilla.org/en-US/docs/Web/API/FormData/FormData)


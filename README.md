# Rails-Form-Helpers-Partials
Rails-Form-Helpers-Partials


# Rails: The CUD of CRUD

### Learning Objectives
- Discuss forms in rails
- What is a CSRF token?
- Use form helpers to generate forms in rails
- Use partials to DRY up our views

## Setting up our rails app

Fork and clone this repo and follow these steps:
- run `bundle install`
- run `rails db:create`
- run `rails db:migrate`
- run `rails db:seed`
- run `rails s` and visit `localhost:3000`

## Routes

Look at `rails routes` after adding a `cats` resources
```
cats      GET    /cats(.:format)          cats#index
          POST   /cats(.:format)          cats#create
new_cat   GET    /cats/new(.:format)      cats#new
edit_cat  GET    /cats/:id/edit(.:format) cats#edit
cat       GET    /cats/:id(.:format)      cats#show
          PATCH  /cats/:id(.:format)      cats#update
          PUT    /cats/:id(.:format)      cats#update
          DELETE /cats/:id(.:format)      cats#destroy
```

## Creating a new Cat

So far, we've done `index` and `show` methods in our controller. But! We are going to need some others in order to create CRUD. What are we going to need?

A `GET` to `/cats/new` will render the new cat form.  The controller method is `new`.

A `POST` to `/cats` will create a new cat.  The controller method is `create`

## Our Cat form

```html
<form action="/cats">
  <input name="authenticity_token" value="<%= form_authenticity_token %>" type="hidden">

  <input placeholder="name" type="text" name="cat[name]" id="cat_name">

  <br>

  <input placeholder="breed" type="text" name="cat[breed]" id="cat_breed">

  <br>

  <input type="submit" name="commit" value="Create Cat" data-disable-with="Create Cat">
</form>
```

Woah, what's that `form_authenticity_token`??? [Here's an exhaustive answer from StackOverflow](https://stackoverflow.com/questions/941594/understanding-the-rails-authenticity-token). Essentially, it protects those routes from being hit by forged requests.

## Our Cat controller

Before we can do anything with our cat form, we need two new controller methods: `new` and `create`. `new` won't do anything for now -- it'll just send back our `new.html.erb`. `create`, on the other hand, is going to create a new cat!


```rb
def new
  @cat = Cat.new
end

def create
  @cat = Cat.new(cat_params)

  if @cat.save
    redirect_to cat_path(@cat)
  else
    # flash[:errors] = @cat.errors.full_messages # ignore this line for now
    render :new
  end
end

private

def cat_params
  params.require(:cat).permit(:name, :breed)
end
```

We're saying that we expect there to be a `cat` in what we get back from the server, and that cat should have the fields `name` and `breed`.

> Side note: any method that corresponds to a route should be `public`.  All others are `private`

### Using the `form_for` helper

So, this is pretty neat so far, right? We can add cats at will! But there is a _better way_ to write that form. Let's take a crack at that.

```erb
<%= form_for @cat do |f| %>

  <%= f.text_field :name, placeholder: "name" %>

  <br>

  <%= f.text_field :breed, placeholder: "breed" %>

  <br>

  <%= f.submit %>
<% end %>
```

Instead of having to write the form ourselves, with the path and the method and the CSRF token and everything, it's now generated for us with this `form_for` tag! Yay!

### You do

Add a link from the cat index page to the new cat page and vise versa.


## Editing a cat

A `GET` to `/cats/:id/edit` will render the edit cat form.  The controller method is `edit`.

A `PATCH` to `/cats/:id` will update a cat.  The controller method is `update`

To edit a cat we can copy the same form (we can DRY this out later) and add `edit` and `update` methods.

```rb
def edit
  @cat = Cat.find(params[:id])
end

def update
  @cat = Cat.find(params[:id])

  if @cat.update(cat_params)
    redirect_to cat_path(@cat)
  else
    # flash[:errors] = @cat.errors.full_messages # ignore this line for now
    render :edit
  end
end
```

### You do

Add a link from the cat show page to the cat edit page and vise versa.

## Deleting a cat (😿)

When we delete a cat, we can still use the `form_for` helper -- we just need to give it some additional information.

In the show view:

```erb
<%= form_for @cat, html: {method: "delete"} do |f| %>
  <%= f.submit "Delete #{@cat.name}" %>
<% end %>
```

We're telling the `form_for` helper that in the HTML for this particular form, the method should be `delete`.

We also have a short hand for the form above:

```erb
<%= button_to "Delete #{@cat.name}", cat_path(@cat), method: :delete %>
```

This will take us to a `destroy` method in the controller.  It can look something like this:

```rb
def destroy
  cat = Cat.find(params[:id])
  cat.destroy
  redirect_to cats_path
end
```

## DRYing up our forms!

Take a look at our edit and new views. That form looks pretty similar, right? Let's abstract it out into a PARTIAL!

We're going to make a new file, `_form.html.erb`, within the `views/cats` directory. Then, we're going to take the form and put it in there.

Then, any time we want to include a form, we can just include this line in our erb file:

```html
<%= render partial: 'form' %>
```

## Validating models

Active Record supports many types of [validations](https://guides.rubyonrails.org/active_record_validations.html).  If, for example, we want to only save models with certain fields present, we can use `validates_presence_of`.


For example, if we only want to save cats that have a `name` and `breed` present, we can add the following:

```rb
class Cat < ApplicationRecord
  validates_presence_of :name, :breed
end
```

This way, cats will not save (create or update) if the `name` or `breed` column is empty.

When trying to call `cat.save`, for example, on a cat with no name.  We will see errors on `cat.errors.full_messages`

## Adding flash messages

It would be great to display a message when the submit is not successful so the user knows what went wrong.  To do this, Rails gives us [`flash`](https://api.rubyonrails.org/classes/ActionDispatch/Flash.html).  `flash` is just a hash-like object that we can add any messages to.

We can display anything in flash, for example, in `app/views/layouts/application.html`.  Everything here shows up on (almost) every page.

```erb
<body>
  <% if flash[:errors] %>
    <div class="flash-errors">
      <%= flash[:errors].join(', ') %>
    </div>
  <% end %>

  <%= yield %>
</body>
```

> You can add some styling to `.flash-errors` in a scss file

`flash` persists in the session just for one request.  This means if we add something to `flash` it will only display for the very next request.

Now we can uncomment the lines that add messages to `flash[:errors]`.  See the errors appear!

### You do

* When a cat is deleted in our controller, set `flash[:message]` to `"You deleted a cat!"`
* Inside of `application.html.erb` add a `div` with the class `flash-message`.  Its content will be `flash[:message]`.
* Style the `div` so the text appears green and bold
* Delete a cat from your app and see a message appear!

## Resources

* [Form helpers](https://guides.rubyonrails.org/form_helpers.html)
* [AR validations](https://guides.rubyonrails.org/active_record_validations.html)
* [Flash](https://guides.rubyonrails.org/action_controller_overview.html#the-flash)
* [Great walkthrough of CRUD app](https://edgeguides.rubyonrails.org/getting_started.html#getting-up-and-running)

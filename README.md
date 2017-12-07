# Rails: The CUD of CRUD

### Learning Objectives
- Discuss forms in rails
- What is a CSRF token?
- Use form helpers to generate forms in rails
- Use partials to DRY up our views

## Setting up our rails app

Fork and clone this repo and follow these steps:
- cd into `CatApp`
- run `bundle install`
- run `rails db:create`
- run `rails db:migrate`
- run `rails db:seed`
- run `rails s` and visit `localhost:3000`

# Creating a new Cat

So far, we've done `index` and `show` methods in our controller. But! We are going to need some others in order to create CRUD. What are we going to need? What do we get when we run `rails routes`?

## Our Cat form

```html
<form action="/cats" method="post">
  <input name="authenticity_token" value="<%= form_authenticity_token %>" type="hidden">
  <input name="name" type="text" placeholder="name">
  <input name="breed" type="text" placeholder="breed">
  <button type="submit">Submit</button>
</form>
```

Woah, what's that `form_authenticity_token`??? [Here's an exhaustive answer from StackOverflow](https://stackoverflow.com/questions/941594/understanding-the-rails-authenticity-token). Essentially, it protects those routes from being hit by forged requests.

## Our Cat controller

Before we can do anything with our cat form, we need two new controller methods: `new` and `create`. `new` won't do anything for now -- it'll just send back our `new.html.erb`. `create`, on the other hand, is going to create a new cat!

```rb
def create
  @cat = Cat.new(name: params[:name], breed: params[:breed])
  if @cat.save
    redirect_to cat_path(@cat)
  else
    render :new
  end
end
```

But, wouldn't it be better to check out our params first before we create the cat? We know what a cat needs to have in order to be created. Let's adjust that.

```rb
def create
  @cat = Cat.new(cat_params)
  if @cat.save
    redirect_to cat_path(@cat)
  else
    render :new
  end
end

private
def cat_params
  params.require(:cat).permit(:name, :breed)
end
```

We're saying that we expect there to be a `cat` in what we get back from the server, and that cat should have the fields `name` and `breed`.

### Using the `form_for` helper

So, this is pretty neat so far, right? We can add cats at will! But there is a _better way_ to write that form. Let's take a crack at that.

```erb
<%= form_for @cat do |f| %>
  <%= f.text_field :name, placeholder: "Name" %>
  <%= f.text_field :breed, placeholder: "Breed" %>
  <%= f.submit "Add new cat"%>
<% end %>
```

Instead of having to write the form ourselves, with the path and the method and the CSRF token and everything, it's now generated for us with this `form_for` tag! Yay!

We also have to add to our `new` method:

```rb
def new
  @cat = Cat.new
end
```

This creates a blank `cat` object that we're going to pass into our view.

## 🚀 LAB!

Add create functionality into your version of the cat app.

# Updating an existing cat

Same as before, just in our `edit` method we have to pass in the cat we're editing.

## 🚀 LAB!

Add update functionality into your version of the cat app.

# Deleting a cat (😿)

When we delete a cat, we can still use the `form_for` helper -- we just need to give it some additional information.

In the show view:

```html
<%= form_for @cat, html: {method: "delete"} do |f| %>
  <%= f.submit "Delete #{@cat.name}???" %>
<% end %>
```

We're telling the `form_for` helper that in the HTML for this particular form, the method should be `delete`.

## 🚀 LAB!

Add delete functionality into your version of the cat app.

# DRYing up our forms!

Take a look at our edit and new views. That form looks pretty similar, right? Let's abstract it out into a PARTIAL!

We're going to make a new file, `_form.html.erb`, within the `views/cats` directory. Then, we're going to take the form and put it in there.

Then, any time we want to include a form, we can just include this line in our erb file:

```html
<%= render partial: 'form'%>
```

Easy peasy!

## 🚀 LAB!

Make a form partial.

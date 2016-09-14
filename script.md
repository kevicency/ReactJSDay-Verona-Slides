Hey, my name is Kevin and welcome to my lightning talk
'CSS id dead! Long live CSS! but in modules please'

Let me ask you a question first: 
'What makes working with CSS so hard?'.
And the answer is not 'Internet Explorer 7'
but it's the globals.

They are basically everywhere in CSS land.

And as we all know, a global state makes developers sad!

And there is no real solution to that problem.
There are Architecture Guidelines for CSS that some of you might know
and Preprocessors that some of you might use.
But they do not solve the problem of global state,
they only make it more manageable.

And the funny thing is that JS had the same problem
a few years ago. But there are no sad JS developers
here today, right?

So what killed the global state in JS?

Well, the modules did! With npm and webpack doing the heavy lifting.

So why don't we just apply the concept of JS modules to CSS.
So we start by installing our css module, f.e. bootstrap, and import 
the stylesheet that comes with it as module.

The difference here is that by importing a stylesheet as a module we 
get a local reference to it. And let's say that all the class names
in this stylesheet module are local by default too.
But the problem still is that CSS only knows global, so how can we safely
get there?

Well, let's see what a stylesheet module exactly is. The stylesheet module
is a map of a local class names defined in the stylesheet, f.e. btn, to a global class name, like that
has you're seeing here. And by having random hashes as global class names
the chances for collisions  are negligible. And in the end, the stylesheet that
will be added to the document will be a *generated* one, 
with all **local** class names replaced by their **global** equivalent.

So how can we actually put this to use?
First, we need to setup wepack by adding a loader definition for css files
and enabling css module support. And that is basically it.

Now, let's create a simple button component that uses CSS modules. here's a reference
to the stylesheet module again. Then, in our component we use the two local class names
required for creating a primary button, to get the corresponding global class names from
the stylesheet module. And when we render this button, the result will look like this.

Now you might be thinking. Well that's nice for production but is there a human readable
version for that during development?

And that is indeed possible. The css loader supports customization of the generated global class
name by setting the localIdentName parameter. In this example, the construct the global class name
by concatenating the path and name of the css file as well as the local class name. And that's the output.

Now I hope you're thinking Well, that's awesome, but what about unit testing? Does that mean
that I need to bundle my unit tests?! And the answer to that is No!. There is a require hook for 
css modules that allow you to require css modules in node during runtime.
You can also customize the generated global class name and if you set this to just the local name
you probably don't event need to change a single thing in you unit tests.

What's still missing is how you get the generated stylesheet into the document. 
You can use the style loader which will render the stylesheet as inline ``<style />`` tag
or you can additionally use the extract-text-webpack-plugin to write the stylesheet to a file.



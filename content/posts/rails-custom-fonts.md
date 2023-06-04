---
title: "Elevate Your Rails 6 Application with Custom Fonts Using Webpacker"
date: 2020-12-16T16:14:59-04:00
draft: false
---

Adding custom fonts to your application can greatly enhance its visual appeal and make a bold statement. In this article, we will explore how to seamlessly import custom fonts into your Rails 6 Application with Webpacker. By the end of this tutorial, you'll be able to effortlessly integrate custom fonts and give your application a unique touch.

## Steps to Import Custom Fonts

1. Download the Font:
Begin by downloading your desired font from a remote location. This could be a font repository or a trusted source like Font Meme.

2. Create a Fonts Folder:
In your Rails project, navigate to the `app/javascript` directory and create a new folder called `fonts`. This is where we will store our custom font files.

3. Create a Fonts SCSS File:
Within the `fonts` folder, create a new SCSS file named `fonts.scss`. This file will be responsible for importing your fonts, including any external URLs if necessary.

Example: `app/javascript/stylesheets/fonts/fonts.scss`
```scss
@import url('../fonts/harringt.ttf');
```

4. Specify the Font Family:
Open your application's CSS file, such as `application.css` or a specific view's CSS file, and define the font family you want to use.

Example: `app/javascript/stylesheets/pages.css`
```css
.page-titles {
  font-family: 'Harrington', cursive;
}
```

5. Apply the Font to Your Application:
To display the custom font, add the corresponding CSS class to the sections or elements where you want it to be used. Alternatively, you can apply the font to the entire body.

Example: `app/views/pages/index.html.erb`
```erb
<!-- pages/index.html.erb -->

<h1 class="page-titles">Custom Font</h1>
```

## Conclusion: Unleash the Power of Custom Fonts
By following these simple steps, you can effortlessly import custom fonts into your Rails 6 Application using Webpacker. This allows you to infuse your application with personality and make a lasting impression on your users.

_Special thanks to [Font Meme](https://fontmeme.com/) for providing the [Harrington](https://fontmeme.com/fonts/harrington-font/#previewtool) font used in this example._


### Source
My own dev.to [post](https://dev.to/gokucodes/adding-custom-fonts-to-rails6-using-webpacker-d51)
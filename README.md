## Moonwalk - a fast and minimalistic blog theme with clean dark mode

- Original project/docs: https://github.com/abhinavs/moonwalk

### Project Structure

    _site/                          Auto generated site.
    assets/css/main.css             -
    _sass/                          -

    bin/                            -
        start                       Startup script to launch locally.

    index.md                        Main homepage.
    translations.md                 Translated articles.
    sysadmin.md                     Sysadmin articles.

    _layouts/
        base.html                   Common HTML structure shared by other layouts.
        navbar.html                 Layout that contains a navbar header/footer.
        translation.html            Layout for arabic/english posts (bidi supported).
        blogpost.html               Layout for english posts.

    _includes/
        head.html                   Header for all pages.
        head_translation.html       -
        translation.html            Card where top half is arabic and bottom half is english.
        card.html                   Card where top half is a hyperlinked title and bottom half is a description.
        post.html                   Card where top half is a hyperlinked title and bottom half is a date.
        list_horizontal.html        Used for footer and header of homepage.
        date_and_social_share.html  -
        toggle_theme_button.html    -
        toggle_theme_js.html        -

## Acknowledgement

This theme's original base is [no style please!](https://github.com/riggraz/no-style-please) theme created by [Riccardo Graziosi](https://riggraz.dev/) - many thanks to him for creating a wonderful theme with nearly no css. 

## License

The theme is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

# Emscripten Browser Cursor Library

Header-only C++ library for easy manipulation of browser mouse pointer cursors, for Emscripten applications.  Single header, standard modern C++ with no dependencies other than Emscripten.


## Functionality

```cpp
enum class cursor                                                     // type-safe enumeration of standard cursor types

bool emscripten_browser_cursor::is_set()                              // returns whether the cursor is currently set

std::optional<cursor> emscripten_browser_cursor::get()                // returns optional representing current cursor, or std::nullopt
std::string emscripten_browser_cursor::get_string()                   // return the currently set cursor as a string value

void emscripten_browser_cursor::set(cursor new_cursor)                // set a new cursor from a cursor enum
void emscripten_browser_cursor::set(std::optional<cursor> new_cursor) // set a new cursor from an optional (passing std::nullopt unsets)
void emscripten_browser_cursor::set(std::string const &new_cursor)    // set a new cursor from an arbitrary string value

void unset();                                                         // clear the current cursor setting
```

## Cursor enums

This library can set any standard CSS cursor style - for a visual reference, see https://developer.mozilla.org/en-US/docs/Web/CSS/cursor.

An enumerated type exists for each of the standard styles, in the format `emscripten_browser_cursor::cursor::nwse_resize`.  Dashes in the standard names correspond to underscores in the enum names.

Two special cases exist - the enum corresponding to `auto` is named `cursor::cursor_auto` and likewise for `default` / `cursor::cursor_default`, to avoid collisions with the equivalent C++ keywords.


## Setting the cursor

The primary use case of this library is to change the currently displayed cursor to match something in your application state, so the set functions are all you'll need most of the time.  They provide different ways to achieve the same thing:

### set(cursor)

Set the current cursor to one of the standard types using an enum:

```cpp
void emscripten_browser_cursor::set(cursor new_cursor)                // set a new cursor from a cursor enum
```

Example:

```cpp
emscripten_browser_cursor::set(emscripten_browser_cursor::cursor::help); // set the "help" cursor
```

### set(std::optional<cursor>)

As above, but accepts a `std::optional` which may have no value.  If an empty optional (`std::nullopt`) is provided, it unsets the current cursor setting.  Note that unset is a different state from set to "default".

```cpp
void emscripten_browser_cursor::set(std::optional<cursor> new_cursor) // set a new cursor from an optional (passing std::nullopt unsets)
```

This is especially useful if you wish to store a state where a custom cursor may or may not be set.  Example:

```cpp
std::optional<emscripten_browser_cursor::cursor> new_cursor;
if(show_help) {
  new_cursor = emscripten_browser_cursor::cursor::help;
} elif(edit_text) {
  new_cursor = emscripten_browser_cursor::cursor::text;
}
emscripten_browser_cursor::set(cursor); // if show_help and edit_text are both false, this will unset the custom cursor, returning it to default behaviour
```

### set(std::string)

In cases where you want to set a cursor as an arbitrary value, you can use the `std::string` overload:

```cpp
void emscripten_browser_cursor::set(std::string const &new_cursor)    // set a new cursor from an arbitrary string value
```

This has a bit overhead due to copying string content, but can allow advanced usage such as specifying a list of URLs - see the section on custom cursors below.

Example:

```cpp
emscripten_browser_cursor::set("progress"); // set the "progress" cursor as a string
```

### unset()

```cpp
void unset();                                                         // clear the current cursor setting
```

Unsets the current cursor setting, if there is one, returning the state to default with no cursor specified.  Note that unset is a fundamentally different state from being set to "auto" or the "default" cursors, although a given browser might behave the same way for each mode.


## Querying current cursor settings

Most users of the library will want to keep track of cursor state in their own application, which will be more efficient than querying the cursor state from the browser.  These functions are provided for convenience, but are relatively costly due to the requirements of string copying and string comparisons.

### is_set()

```cpp
bool emscripten_browser_cursor::is_set()                              // returns whether the cursor is currently set
```

### get()

```cpp
std::optional<cursor> emscripten_browser_cursor::get()                // returns optional representing current cursor, or std::nullopt
```

### get_string()

```cpp
std::string emscripten_browser_cursor::get_string()                   // return the currently set cursor as a string value
```

## Using with custom cursors

You can easily instruct the library to load one or more images from URLs, by setting strings as described at https://developer.mozilla.org/en-US/docs/Web/CSS/cursor.  Don't forget to provide a fallback from the standard cursor types.  For example:

```cpp
emscripten_browser_cursor::set("url('http://my_image_url.png'), progress"); // set an image from a URL, wth a fallback to the "progress" cursor.
```

## Performance tips

The most efficient way to use this library is to just call `set()` with the `cursor` enums, avoiding the string functions.  Keep track of last set cursor state locally in your application, and only call a set function if the state has changed.

Avoid calling the getter functions on a frame by frame basis, only using them if there is a possibility that cursor state has been changed by another part of the page other than your canvas.

## Using with ImGui

ImGui attempts to modify cursors by default - it's possible to tie in this library with ImGui for easy cursor changes in-browser.  The following is all that's needed, after you've finished rendering all other ImGui windows:

```cpp
if(ImGui::GetIO().WantCaptureMouse) {
    switch(ImGui::GetMouseCursor()) {
    case ImGuiMouseCursor_Arrow:
      emscripten_browser_cursor::set(emscripten_browser_cursor::cursor::cursor_default);
      break;
    case ImGuiMouseCursor_TextInput:                                              // When hovering over InputText, etc.
      emscripten_browser_cursor::set(emscripten_browser_cursor::cursor::text);
      break;
    case ImGuiMouseCursor_ResizeAll:                                              // (Unused by Dear ImGui functions)
      emscripten_browser_cursor::set(emscripten_browser_cursor::cursor::move);
      break;
    case ImGuiMouseCursor_ResizeNS:                                               // When hovering over a horizontal border
      emscripten_browser_cursor::set(emscripten_browser_cursor::cursor::ns_resize);
      break;
    case ImGuiMouseCursor_ResizeEW:                                               // When hovering over a vertical border or a column
      emscripten_browser_cursor::set(emscripten_browser_cursor::cursor::ew_resize);
      break;
    case ImGuiMouseCursor_ResizeNESW:                                             // When hovering over the bottom-left corner of a window
      emscripten_browser_cursor::set(emscripten_browser_cursor::cursor::nesw_resize);
      break;
    case ImGuiMouseCursor_ResizeNWSE:                                             // When hovering over the bottom-right corner of a window
      emscripten_browser_cursor::set(emscripten_browser_cursor::cursor::nwse_resize);
      break;
    case ImGuiMouseCursor_Hand:                                                   // (Unused by Dear ImGui functions. Use for e.g. hyperlinks)
      emscripten_browser_cursor::set(emscripten_browser_cursor::cursor::pointer);
      break;
    case ImGuiMouseCursor_NotAllowed:                                             // When hovering something with disallowed interaction. Usually a crossed circle.
      emscripten_browser_cursor::set(emscripten_browser_cursor::cursor::not_allowed);
      break;
    }
  }
```

Because you're not using the native ImGui back-end cursor modification behaviour, it also makes sense to disable the associated functionality during your setup:

```cpp
ImGui::GetIO().ConfigFlags |= ImGuiConfigFlags_NoMouseCursorChange; // disable ImGui attempting to change the cursor itself
ImGui::GetIO().Fonts->Flags |= ImFontAtlasFlags_NoMouseCursors; // don't take up font atlas space with mouse cursor images 
```

## Other useful libraries

You may also find the following Emscripten helper libraries useful:

- [Emscripten Browser File Library](https://github.com/Armchair-Software/emscripten-browser-file) - allows you to transfer files using the browser upload / download interface, into memory in your C++ program.
- [Emscripten Browser Clipboard Library](https://github.com/Armchair-Software/emscripten-browser-clipboard) - easy handling of browser copy and paste events in your C++ code.

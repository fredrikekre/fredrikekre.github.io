
name: titlepage


class: middle, centre






# `display`, `show` and `print`






### How Julia's display system works


Fredrik Ekre, JuliaCon 2020


Slides @ [fredrikekre.se/juliacon](https://fredrikekre.se/juliacon/)


---






# Introduction


--


Julia's display system


  * Powerful but complicated system


--


  * Many functions related to producing output: `display`, `show`, `print`, `write`


--


  * Support for many different output formats/representaions, e.g. text, images, etc.


--


  * Possible to implement "pretty printing" for custom types


---






# Agenda


  * Overview of the general structure and the functions involved (`display` etc)


  * Example how to extend the system for user-defined types


  * Example of how the system is beeing used in the wild


---


name: basic






# From object to output


For an object `x`, the following happens in the REPL


--


  * `display(x)`

    Selects a suitable display from the internal stack


--


  * `display(disp, x)`

    Selects a suitable "MIME"-format for the representation that the display supports


--


  * `show(disp, mime, x)`

    Produces and writes the actual output to the IO buffer, for example using `print` or `write`.


---






# MIME - Multipurpose Internet Mail Extensions


--


  * A system to describe the representation of objects in e.g. email, webpages etc

    ```bash
    $ curl -LI google.com
    HTTP/1.1 200 OK
    Content-Type: text/html; charset=ISO-8859-1
    ...
    ```


--


  * In Julia there is a type `MIME` to represent different formats

    ```julia
    julia> MIME"text/plain"()
    MIME type text/plain

    julia> MIME"image/png"()
    MIME type image/png
    ```


---


name: display






# `display`


  * The `display` function is the top-most function


--


  * Usually a "behind the scenes" function that users don't interact with


--


  * Implemented by "displays" and frontends such as the REPL, editors (e.g. Juno, VSCode), IJulia, etc.


--


<hr>






## Example


```julia
struct REPLDisplay <: AbstractDisplay end

display(d::REPLDisplay, x) = ...
```


---


name: show






# `show`


  * The `show` function produces and writes output in the format specified by the MIME. For example

    ```julia
    show(io, MIME"image/png", x)
    ```

    writes an image (PNG) representation of `x` to `io`.


--


  * Implemented to customize output for user-types, e.g.

      * `show(io::IO, ::MIME"text/plain", ::MyType)` for plain text representation
      * `show(io::IO, ::MIME"text/html", ::MyType)` for HTML representation
      * `show(io::IO, ::MIME"image/png", ::MyType)` for PNG representation
      * `show(io::IO, ::MIME"application/json", ::MyType)` for JSON representation
      * `show(io::IO, ::MIME"application/pdf", ::MyType)` for JSON representation
      * ...


---


name: print-etc






## `print`


--


> Print a canonical (un-decorated) text representation. The representation used by print includes minimal formatting ...



--


  * `print` falls back to `show`
  * Should generally not be extended for user types (extend `show`)


--






## `write`


  * Writes the binary representation of objects


---






# Customization


  * Output options can be passed to `show` using an `IOContext`


--


  * `IOContext` is a dictionary-like object that wraps an IO buffer


--


  * `IOContext` can be queried for output options

      * `:compact`: specifying that small values should be printed more compactly, e.g. fewer digits
      * `:limit`: specifying that containers should be truncated, e.g. showing … in place of most elements
      * `:color`: specifying whether ANSI color/escape codes are supported


--


```julia
function show(io::IO, ::MIME"text/plain", x::MyType)
   if get(io, :color, false)
       # produce color output of x and write to io
   else
       # produce no-color output of x and write to io
   end
end
```


---


name: example




# Example


Define `show` methods for a user defined type: `text/plain`


--


```julia
julia> struct JuliaLogo end
```


--


```julia
julia> x = JuliaLogo();

julia> x
JuliaLogo()
```


--


```julia
function Base.show(io::IO, ::MIME"text/plain", ::JuliaLogo)
    print(io, """   _
     _(_)_
    (_) (_)""")
end
```


--


```julia
julia> x
   _
 _(_)_
(_) (_)

```


---


name: example




# Example


Define `show` methods for a user defined type: `image/png` and `image/svg+xml`


--


.example-image[![](assets/ijulia-1.png)]


---


name: example




# Example


Define `show` methods for a user defined type: `image/png` and `image/svg+xml`


.example-image[![](assets/ijulia-2.png)]


---


name: example




# Example


Define `show` methods for a user defined type: `image/png` and `image/svg+xml`


.example-image[![](assets/ijulia-3.png)]


---


name: ijulia






# Example: `IJulia`


Example of a frontend that uses lots of Julia's display functionality


--


  * Defines a display to show output:

    ```julia
    struct InlineDisplay <: AbstractDisplay end
    ```


--


  * `display` implementation supports multiple MIMEs (text, images, html, ...)


--


  * `IJulia` captures multiple representations of the same object and the frontend makes the choice about what to display


---


name: ijulia




# Example: `IJulia`


Look inside a notebook


Cell #2:


```json
{
 "source": [
  "JuliaLogo()"
 ],
 "outputs": {
   "text/plain": [
    "JuliaLogo()"
   ],
  },
},
```


---


name: ijulia




# Example: `IJulia`


Look inside a notebook


Cell #4:


```json
{
 "source": [
  "JuliaLogo()"
 ],
 "outputs": {
   "image/png": "iVBORw0KGgoAAAANSUhEUgAAAGoAAABiCAYAAAC4ck...",
   "text/plain": [
    "JuliaLogo()"
   ],
  },
},
```


---


class: center, middle






# Thanks for listening!


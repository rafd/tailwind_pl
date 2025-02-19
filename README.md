# Tailwind_pl

Based on [Girouette](https://github.com/green-coder/girouette), Tailwind_pl is a pack for SWI-Prolog that will watch the given directories for changes to Prolog files, search for [TailwindCSS](https://tailwindcss.com/docs)-style CSS selectors and generates a CSS file containing just the used selectors.

This lets you use Tailwind-style classes, but without the overhead of having to pull in an enormous stylesheet with all possible styles.
This also allows more dynamic class-names that, with exact colours or numbers that would otherwise be unreasonable to use.

## Installation

``` prolog
?- pack_install(tailwind_pl).
```

## Example Usage


``` prolog
  :- module(myapp, [go/1]).

:- use_module(library(filesex), [directory_file_path/3, make_directory_path/1]).
:- use_module(library(http/http_dispatch), [http_redirect/3,
                                            http_dispatch/1,
                                            http_reply_file/3,
                                            http_handler/3,
                                            http_location_by_id/2
                                           ]).
:- use_module(library(http/html_write), [html//1,
                                         reply_html_page/2,
                                         html_receive//1]).
:- use_module(library(http/thread_httpd), [http_server/2,
                                           http_stop_server/2]).
:- use_module(library(tailwind), [start_watching_dirs/3,
                                  stop_watching_dirs/1,
                                  reset_style/1]).

%! go(+Port) is det.
%  Interactive entry point to start the server.
go(Port) :-
    start_css_watcher,
    http_server(http_dispatch, [port(Port)]).

user:file_search_path(css, CssDir) :-
    working_directory(Cwd, Cwd),
    directory_file_path(Cwd, "../resources/css", CssDir).

:- dynamic css_watcher/1.
start_css_watcher :-
    stop_css_watcher,
    working_directory(Cwd, Cwd),
    directory_file_path(Cwd, "../resources/css", CssDir),
    ( exists_directory(CssDir) -> true ; make_directory_path(CssDir) ),
    absolute_file_name(css("styles.css"), CssFile, [access(none)]),
    start_watching_dirs([Cwd], CssFile, Watcher),
    assertz(css_watcher(Watcher)).

stop_css_watcher :-
    css_watcher(Watcher), !,
    stop_watching_dirs(Watcher),
    retractall(css_watcher(Watcher)).
stop_css_watcher.

% Routes

:- multifile http:location/3.
:- dynamic http:location/3.

http:location(css, root(css), []).

:- http_handler(root(.), home_page_handler, [method(get), id(home_page),
                                             spawn(handler_pool)]).

:- http_handler(css('styles.css'), css_file_handler, [method(get), id(tw_css)]).
:- http_handler(css('reset.css'), reset_css_handler, [method(get), id(reset_css)]).

home_page_handler(_Request) :-
    reply_html_page(title('Hotwire.pl'), [\home_page]).

css_file_handler(Request) :-
    http_reply_file(css('styles.css'), [], Request).

reset_css_handler(_) :-
    reset_style(Style),
    format("Content-Type: text/css~n~n~s", [Style]).

% content

user:head(default, Head) -->
    html([Head,
          \html_receive(head),
          link([rel(stylesheet), href(#(reset_css))], []),
          link([rel(stylesheet), href(#(tw_css))], [])
         ]).

% Home Page

home_page -->
    html(
        div(class(["flex-col", "text-purple-500", 'bg-rose-50']),
            [h2(class(['text-lg',
                       'hover:animation-ping',
                       'bg-green-300', 'hover:text-rose-100']),
                "Info"),
             \p("here's some stuff, some information"),
             \p("Another paragraph")
            ])
    ).

p(Text) -->
    html(p(class(['text-sm', 'text-blue', 'hover:opacity-50']),
           Text)).
```

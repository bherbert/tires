# tires

**A command line tool for remembering what tires are on my motorcycle.**

[Changelog](#changelog)

## Contents

- [What and why](#what-and-why)
- [Installation](#installation)
- [The "tires" file](#the-tires-file)
- [Configuration](#configuration)
- [Usage](#usage)
- [Extras](#extras)
- [Troubleshooting](#troubleshooting)

<!-- end toc -->

## What and why

`tires` is a basic CLI for adding and listing "what tires are on my motorcycle and how many miles on each" information in a [TaskPaper-formatted](http://www.hogbaysoftware.com/products/taskpaper) text file. It allows for multiple sections/categories and flexible output formatting.

I ride a mix of on- and off-road on a BMW R1200GS Adventure.  When doing an extended on-road trip, I will mount a set of 90/10 (on/off) tires like Metzler Tourances.  When doing more local off-road riding (between big road trips), I will mount a more 50/50 tire like a Heidenau K60 Scout. 

The problem is, sets of tires can be swapped as needed (I own a COATS 220 tire changer), so whatever set just came off might well have a lot of miles left on the tread. But how much tread?  How many miles are actually ON the tires I just took off? 

I'm retired, so I try to get as many miles on a set of tires as I can.  `tires` helps me manage all  that.

NOTE: This project is a shameless rip-off of the awesome little script by Brett Terpstra named "doing" [link](https://github.com/ttscoff/doing/).


## Installation

    $ [sudo] gem install tires

Only use `sudo` if your environment requires it. If you're using the system Ruby on a Mac, for example, it will likely be necessary. If `gem install tires` fails, then run `sudo gem install tires` and provide your administrator password.

Run `tires config` to open your `~/.tiresrc` file in the editor defined in the $EDITOR environment variable. Set up your `tires_file` right away (where you want entries to be stored), and cover the rest after you've read the docs.

See the [support](#support) section below for troubleshooting details.

## The "tires" file

The file that stores all of your entries is generated the first time you add an entry with `tires XX` (or `tires YY`). By default the file is created in "~/what_tires_are_mounted_.md", but this can be modified in the config file.

The format of the "tires" file is TaskPaper-compatible. You can edit it by hand at any time (in TaskPaper or any text editor), but it uses a specific format for parsing, so be sure to maintain the dates and pipe characters. 

Notes are anything in the list without a leading hyphen and date. They belong to the entry directly before them, and they should be indented one level beyond the parent item. When using the `now` and `later` commands on the command line, you can start the entry with a quote and hit return, then type the note and close the quote. Anything after the first line will be turned into a TaskPaper-compatible note for the task and can be displayed in templates using `%note`.

Notes can be prevented from ever appearing in output with the global option `--no-notes`: `tires --no-notes show all`.

#FOLLOWING NEEDS CUSTOM WRITTEN

## Configuration

A basic configuration looks like this:

    ---
    tires_file: /Users/username/Dropbox/nvALT2.2/?? What Tires Are Mounted.md
    current_section: Currently
    default_template: '%date: %title%note'
    default_date_format: '%Y-%m-%d %H:%M'
    views:
      color:
        date_format: '%F %_I:%M%P'
        section: Currently
        count: 10
        wrap_width: 0
        template: '%boldblack%date %boldgreen| %boldwhite%title%default%note'
        order: desc
    templates:
      default:
        date_format: '%Y-%m-%d %H:%M'
        template: '%date | %title%note'
        wrap_width: 0
      today:
        date_format: '%_I:%M%P'
        template: '%date: %title%odnote'
        wrap_width: 0
      last:
        date_format: '%_I:%M%P on %a'
        template: '%title (at %date)%odnote'
        wrap_width: 0
      recent:
        date_format: '%_I:%M%P'
        template: '%date > %title%odnote'
        wrap_width: 50
    :include_notes: true


The config file is stored in "~/.tiresrc", and is created on the first run. 

### Tires file location

The one thing you'll probably want to adjust is the file that the notes are stored in. That's the `tires_file` key:

    tires_file: /Users/username/Dropbox/nvALT2.2/?? What Tires are Mounted.md

### "Current actions" section

You can rename the section that holds your current tasks. By default, this is "Currently," but if you have some other bright idea, feel free:

    current_section: Currently

### Default editors

The setting `editor_app` only applies to Mac OS X users. It's the default application that the command `tires open` will open your WWID file in. If this is blank, it will be opened by whatever the system default is, or you can use `-a app_name` or `-b bundle_id` to override.

In the case of the `tires now -e` command, your $EDITOR environment variable will be used to complete the entry text and notes. Set it in your `~/.bash_profile` or whatever is appropriate for your system:

    export EDITOR="mate -w"

The only requirements are that your editor be launchable from the command line and able to "wait." In the case of Sublime Text and TextMate, just use `-w` like this: `export EDITOR="subl -w"`.

### Templates

The config also contains templates for various command outputs. Include placeholders by placing a % before the keyword. The available tokens are:

- `%title`: the "what was I doing" entry line
- `%date`: the date based on the template's "date_format" setting
- `%shortdate`: a custom date formatter that removes the day/month/year from the entry if they match the current day/month/year
- `%note`: Any note in the entry will be included here, a newline and tabs are automatically added.
- `%odnote`: The notes with a leading tab removed (outdented note)
- `%hr`: a horizontal rule (`-`) the width of the terminal
- `%hr_under`: a horizontal rule (`_`) the width of the terminal
- `%n`: inserts a newline
- `%t`: inserts a tab
- `%[color]`: color can be black, red, green, blue, yellow, magenta, cyan or white
    - you can prefix "bg" to affect background colors (%bgyellow)
    - prefix "bold" and "boldbg" for strong colors (%boldgreen, %boldbgblue)
- `%interval`: when used with the `-t` switch on the `show` command, it will display the time between a timestamp or `@start(date)` tag and the `@done(date)` tag, if it exists. Otherwise, it will remain empty.

Date formats are based on Ruby [strftime](http://www.ruby-doc.org/stdlib-2.1.1/libdoc/date/rdoc/Date.html#method-i-strftime) formatting.

My normal template for the `recent` command looks like this:

    recent:
      date_format: '%_I:%M%P'
      template: '%date > %title%odnote'
      wrap_width: 88

And it outputs:
    
    $ doing recent 3
     4:30am > Made my `console` script smarter...
        Checks first argument to see if it's a file, if it is, that's the log

        Else, it checks the first argument for a ".log" suffix and does a search in the user
        application logs with `find` for it.

        Otherwise, system.log.

        I also made an awesome Cope wrapper for it...
    12:00pm > Working on `doing` again.
    12:45pm > I think this thing (doing) is ready to document and distribute
    $ 

You can get pretty clever and include line breaks and other formatting inside of double quotes. If you want multiline templates, just use "\n" in the template line and after the next run it will be rewritten as proper YAML automatically.

For example, this block:

    recent:
      date_format: '%_I:%M%P'
      template: "\n%hr\n%date\n > %title%odnote\n%hr_under"
      wrap_width: 100

will rewrite to:

    recent:
      date_format: '%_I:%M%P'
      template: |2-

        %hr
        %date
         > %title%odnote
        %hr_under
      wrap_width: 100

and output my recent entries like this:

    $ doing recent 3
    -----------------------------------------------------------------------
     4:30am
     > Made my `console` script smarter...
        Checks first argument to see if it's a file, if it is, that's the log

        Else, it checks the first argument for a ".log" suffix and does a search in the user application
        logs with `find` for it.

        Otherwise, system.log.

        I also made an awesome Cope wrapper for it...
    _______________________________________________________________________

    -----------------------------------------------------------------------
    12:00pm
     > Working on `doing` again.
    _______________________________________________________________________

    -----------------------------------------------------------------------
    12:45pm
     > I think this thing (doing) is ready to document and distribute
    _______________________________________________________________________

    $ 

### Custom views

You can create your own "views" in the `~/.tiresrc` file and view them with `tires view view_name`. Just add a section like this:

    views:
      old:
        section: Old
        count: 5
        wrap_width: 0
        date_format: '%F %_I:%M%P'
        template: '%date | %title%note'
        order: asc
        tags: done finished cancelled
        tags_bool: ANY

You can add additional custom views, just nest them under the "views" key (indented two spaces from the edge). Multiple views would look like this:

    views:
      later:
        section: Later
        count: 5
        wrap_width: 60
        date_format: '%F %_I:%M%P'
        template: '%date | %title%note'
      old:
        section: Old
        count: 5
        wrap_width: 0
        date_format: '%F %_I:%M%P'
        template: '%date | %title%note' 

The "section" key is the default section to pull entries from. Count and section can be overridden at runtime with the `-c` and `-s` flags. Setting `section` to All will combine all sections in the output.

You can add new sections with `tires add_section section_name`. You can also create them on the fly by using the `-s section_name` flag when running `tires now`. For example, `tires now -s Misc just a random side note` would create the "just a random side note" entry in a new section called "Misc," if Misc didn't already exist.

The `tags` and `tags_bool` keys allow you to specify tags that the view is filtered by. You can list multiple tags separated by spaces, and then use `tags_bool` to specify "ALL," "ANY," or "NONE" to determine how it handles the multiple tags.

The `order` key defines the sort order of the output. This is applied _after_ the tasks are retrieved and cut off at the maximum number specified in `count`.

Regarding colors, you can use them to create very nice displays if you're outputting to a color terminal. Example:

    color:
      date_format: '%F %_I:%M%P'
      section: Currently
      count: 10
      wrap_width: 0
      template: '%boldblack%date %boldgreen| %boldwhite%title%default%note'

Outputs: 

![](http://ckyp.us/XKpj+)

## Usage

    tires [global options] command [command options] [arguments...]

### Global options:

    --[no-]notes        - Output notes if included in the template (default: enabled)
    --version           - Display the program version
    --help              - Show help message and usage summary

### Commands:

    help           - Shows a list of commands and global options
    help [command] - Shows help for any command (`tires help now`)

#### Adding entries:

    now      - Add an entry
    later    - Add an item to the Later section
    done     - Add a completed item with @done(date). No argument finishes last entry.
    meanwhile - Finish any @meanwhile tasks and optionally create a new one

The `tires now` command can accept `-s section_name` to send the new entry straight to a non-default section. It also accepts `--back AMOUNT` to let you specify a start date in the past using "natural language." For example, `tires now --back 25m ENTRY` or `tires now --back "yesterday 3:30pm" ENTRY`.

You can finish the last unfinished task when starting a new one using `tires now` with the `-f` switch. It will look for the last task not marked `@done` and add the `@done` tag with the start time of the new task (either the current time or what you specified with `--back`).

`tires done` is used to add an entry that you've already completed. Like `now`, you can specify a section with `-s section_name`. You can also skip straight to Archive with `-a`.

`tires done` can also backdate entries using natural language with `--back 15m` or `--back "3/15 3pm"`. That will modify the starting timestamp of the entry. You can also use `--took 1h20m` or `--took 1:20` to set the finish date based on a "natural language" time interval. If `--took` is used without `--back`, then the start date is adjusted (`--took` interval is subtracted) so that the completion date is the current time.

When used with `tires done`, `--back` and `--took` allow time intervals to be accurately counted when entering items after the fact. `--took` is also available for the `tirese finish` command, but cannot be used in conjunction with `--back`. (In `finish` they both set the end date, and neither has priority. `--back` allows specific days/times, `--took` uses time intervals.)

All of these commands accept a `-e` argument. This opens your command line editor as defined in the environment variable `$EDITOR`. Add your entry, save the temp file and close it, and the new entry will be added. Anything after the first line is included as a note on the entry.

`tires meanwhile` is a special command for creating and finishing tasks that may have other entries come before they're complete. When you create an entry with `tires meanwhile [entry text]`, it will automatically complete the last @meanwhile item (dated @done tag) and add the @meanwhile tag to the new item. This allows time tracking on a more general basis, and still lets you keep track of the smaller things you do while working on an overarching project. The `meanwhile` command accepts `--back [time]` and will backdate the @done tag and start date of the new task at the same time. Running `meanwhile` with no arguments will simply complete the last @meanwhile task. See `tires help meanwhile` for more options.

#### Modifying entries:

    finish      - Mark last X entries as @done
    tag         - Tag last entry
    note        - Add a note to the last entry

`tires finish` by itself is the same as `tires done` by itself. It adds `@done(timestamp)` to the last entry. It also accepts a numeric argument to complete X number of tasks back in history. Add `-a` to also archive the affected entries.

`tires finish` also provides an `--auto` flag, which you can use to set the end time of any entry to 1 minute before the start time of the next. Running a command such as `tires finish --auto 10` will go through the last 10 entries and sequentially update any without a `@done` tag with one set to the time just before the next entry in the list.

As mentioned above, `finish` also accepts `--back "2 hours"` (sets the finish date from time now minus interval) or `--took 30m` (sets the finish date to time started plus interval) so you can accurately add times to completed tasks, even if you don't do it in the moment.

`tag` adds one or more tags to the last entry, or specify a count with `-c X`. Tags are specified as basic arguments, separated by spaces. For example:

    tires tag -c 3 client cancelled

... will mark the last three entries as "@client @cancelled." Add `-r` as a switch to remove the listed tags instead.

`note` lets you enter a note on the last entry. You can specify a section with `-s section_name`. `-e` will open your $EDITOR for typing the note, but you can just include it on the command line after any arguments. You can also pipe a note in on STDIN (`echo "fun stuff"|tires note`). The `-r` switch will remove/replace a note; if there's new note text passed when using the `-r` switch, it will replace any existing note. If the `-r` switch is used alone, any existing note will be removed.

You can also add notes at the time of entry by using the `-n` or `--note` flag with `tires now`, `tires later`, or `tires done`. If you pass in text to any of the creation commands that has multiple lines, everything after the first line break will become the note.

#### Displaying entries:

    show      - List all entries
    recent    - List recent entries
    today     - List entries from today
    yesterday - List entries from yesterday
    last      - Show the last entry
    grep      - Show entries matching text or pattern

`tires show` on its own will list all entries in the "Currently" section. Add a section name as an argument to display that section instead. Use "all" to display all entries from all sections.

You can filter the `show` command by tags. Simply list them after the section name (or "all"). The boolean defaults to "ANY," meaning any entry that contains any of the listed tags will be shown. You can use `-b ALL` or `-b NONE` to change the filtering behavior: `tires show all done cancelled -b NONE` will show all tasks from all sections that do not have either "@done" or "@cancelled" tags.

Use `-c X` to limit the displayed results. Combine it with `-a newest` or `-a oldest` to choose which chronological end it trims from. You can also set the sort order of the output with `-s asc` or `-s desc`.

The `show` command can also show the time spent on a task if it has a `@done(date)` tag with the `-t` option. This requires that you include a `%interval` token in template -> default in the config. You can also include `@start(date)` tags, which override the timestamp when calculating the intervals.

If you have a use for it, you can use `--csv` on the show or view commands to output the results as a comma-separated CSV to STDOUT. Redirect to a file to save it: `tires show all done --csv > ~/Desktop/done.csv`.

`tires yesterday` is great for stand-ups, thanks to [Sean Collins](https://github.com/sc68cal) for that. Note that you can show yesterday's activity from an alternate section by using the section name as an argument (e.g. `tires yesterday archive`).

#### Views

    view     - Display a user-created view
    views    - List available custom views

Display any of the custom views you make in `~/.tiresrc` with the `view` command. Use `tires views` to get a list of available views. Any time a section or view is specified on the command line, fuzzy matching will be used to find the closest match. Thus, `lat` will match `Later`, etc..

#### Sections

    sections    - List sections
    choose      - Select a section to display from a menu
    add_section - Add a new section to the "tires" file

#### Utilities

    archive  - Move entries between sections
    open     - Open the "tires" file in an editor (OS X)
    config   - Edit the default configuration

#### Archiving

    COMMAND OPTIONS
        -k, --keep=arg - Count to keep (ignored if archiving by tag) (default: 5)
        -t, --to=arg   - Move entries to (default: Archive)
        -b, --bool=arg - Tag boolean (default: AND)

The `archive` command will move entries from one section (default: Currently) to another section (default: Archive). 

`tires archive` on its own will move all but the most recent 5 entries from currently into the archive.

`tires archive other_section` will archive from "other_section" to Archive.

`tires archive other_section -t alternate` will move from "other_section" to "alternate." You can use the `-k` flag on any of these to change the number of items to leave behind. To move everything, use `-k 0`.

You can also use tags to archive. You define the section first, and anything following it is treated as tags. If your first argument starts with "@", it will assume all sections and assume any following arguments are tags.

By default tag archiving uses an "AND" boolean, meaning all the tags listed must exist on the entry for it to be moved. You can change this behavior with `-b OR` or `-b NONE` ("ALL" and "ANY" also work). 

Example: Archive all Currently items for @client that are marked @done

    tires archive @client @done

---

## Troubleshooting

### Errors after "Successfully installed..."

If you get errors in the terminal immediately after a message like:

    Successfully installed tires-x.x.x
    2 gems installed

...it may just be documentation related. If running `tires` works, you can ignore them. If not, try running the install command again with `--no-document`:

    $ gem install --no-document tires

### Command not found

If running `tires` after a successful install gives you a "command not found" error, then your gem path isn't in your $PATH, meaning the system can't find it. To locate the gem and link it into your path, you can try this:

    cd $GEM_PATH/bin
    ln -s tires /usr/local/bin/

Then try running `tires` and see if it works.

### Encoding errors

Ruby is rife with encoding inconsistencies across platforms and versions. Feel free to file issues (see below).

### Support

I'm not making any money on `tires`, and I don't plan to spend a lot of time fixing errors on an array of operating systems and platforms I don't even have access to. You'll probably have to solve some things on your own.

That said, you can get support from other users (and occasionally me) on GitHub. If you run into a replicatable bug in your environment, please [post an issue](https://github.com/bherbert/tires/issues) and include your platform, OS version, and the result of `ruby -v`, along with a copy/paste of the error message. To get a more verbose error message, try running `GLI_DEBUG=true tires [...]` for a full trace.

### Changelog

#### 0.1.0

- basic stuff


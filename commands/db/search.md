# wp db search

Finds a string in the database.

Searches through all of the text columns in a selection of database tables for a given string, Outputs colorized references to the string.

Defaults to searching through all tables registered to $wpdb. On multisite, this default is limited to the tables for the current site.

### OPTIONS

&lt;search&gt;
: String to search for. The search is case-insensitive by default.

[&lt;tables&gt;...]
: One or more tables to search through for the string.

[\--network]
: Search through all the tables registered to $wpdb in a multisite install.

[\--all-tables-with-prefix]
: Search through all tables that match the registered table prefix, even if not registered on $wpdb. On one hand, sometimes plugins use tables without registering them to $wpdb. On another hand, this could return tables you don't expect. Overrides --network.

[\--all-tables]
: Search through ALL tables in the database, regardless of the prefix, and even if not registered on $wpdb. Overrides --network and --all-tables-with-prefix.

[\--before_context=&lt;num&gt;]
: Number of characters to display before the match.
\---
default: 40
\---

[\--after_context=&lt;num&gt;]
: Number of characters to display after the match.
\---
default: 40
\---

[\--regex]
: Runs the search as a regular expression (without delimiters). The search becomes case-sensitive (i.e. no PCRE flags are added). Delimiters must be escaped if they occur in the expression. Because the search is run on individual columns, you can use the `^` and `$` tokens to mark the start and end of a match, respectively.

[\--regex-flags=&lt;regex-flags&gt;]
: Pass PCRE modifiers to the regex search (e.g. 'i' for case-insensitivity).

[\--regex-delimiter=&lt;regex-delimiter&gt;]
: The delimiter to use for the regex. It must be escaped if it appears in the search string. The default value is the result of `chr(1)`.

[\--table_column_once]
: Output the 'table:column' line once before all matching row lines in the table column rather than before each matching row.

[\--one_line]
: Place the 'table:column' output on the same line as the row id and match ('table:column:id:match'). Overrides --table_column_once.

[\--matches_only]
: Only output the string matches (including context). No 'table:column's or row ids are outputted.

[\--stats]
: Output stats on the number of matches found, time taken, tables/columns/rows searched, tables skipped.

[\--table_column_color=&lt;color_code&gt;]
: Percent color code to use for the 'table:column' output. For a list of available percent color codes, see below. Default '%G' (bright green).

[\--id_color=&lt;color_code&gt;]
: Percent color code to use for the row id output. For a list of available percent color codes, see below. Default '%Y' (bright yellow).

[\--match_color=&lt;color_code&gt;]
: Percent color code to use for the match (unless both before and after context are 0, when no color code is used). For a list of available percent color codes, see below. Default '%3%k' (black on a mustard background).

[\--fields=&lt;fields&gt;]
: Get a specific subset of the fields.

[\--format=&lt;format&gt;]
: Render output in a particular format.
\---
options:
  - table
  - csv
  - json
  - yaml
  - ids
  - count
\---

The percent color codes available are:

| Code | Color
| ---- | -----
|  %y  | Yellow (dark) (mustard)
|  %g  | Green (dark)
|  %b  | Blue (dark)
|  %r  | Red (dark)
|  %m  | Magenta (dark)
|  %c  | Cyan (dark)
|  %w  | White (dark) (light gray)
|  %k  | Black
|  %Y  | Yellow (bright)
|  %G  | Green (bright)
|  %B  | Blue (bright)
|  %R  | Red (bright)
|  %M  | Magenta (bright)
|  %C  | Cyan (bright)
|  %W  | White
|  %K  | Black (bright) (dark gray)
|  %3  | Yellow background (dark) (mustard)
|  %2  | Green background (dark)
|  %4  | Blue background (dark)
|  %1  | Red background (dark)
|  %5  | Magenta background (dark)
|  %6  | Cyan background (dark)
|  %7  | White background (dark) (light gray)
|  %0  | Black background
|  %8  | Reverse
|  %U  | Underline
|  %F  | Blink (unlikely to work)

They can be concatenated. For instance, the default match color of black on a mustard (dark yellow) background `%3%k` can be made black on a bright yellow background with `%Y%0%8`.

### AVAILABLE FIELDS

These fields will be displayed by default for each result:

* table
* column
* match
* primary_key_name
* primary_key_value

### EXAMPLES

    # Search through the database for the 'wordpress-develop' string
    $ wp db search wordpress-develop
    wp_options:option_value
    1:http://wordpress-develop.dev
    wp_options:option_value
    1:https://example.com/foo
        ...

    # Search through a multisite database on the subsite 'foo' for the 'example.com' string
    $ wp db search example.com --url=example.com/foo
    wp_2_comments:comment_author_url
    1:https://example.com/
    wp_2_options:option_value
        ...

    # Search through the database for the 'https?://' regular expression, printing stats.
    $ wp db search 'https?://' --regex --stats
    wp_comments:comment_author_url
    1:https://wordpress.org/
        ...
    Success: Found 99146 matches in 10.752s (10.559s searching). Searched 12 tables, 53 columns, 1358907 rows. 1 table skipped: wp_term_relationships.

    # SQL search database table 'wp_options' where 'option_name' match 'foo'
    wp db query 'SELECT * FROM wp_options WHERE option_name like "%foo%"' --skip-column-names
    +----+--------------+--------------------------------+-----+
    | 98 | foo_options  | a:1:{s:12:"_multiwidget";i:1;} | yes |
    | 99 | foo_settings | a:0:{}                         | yes |
    +----+--------------+--------------------------------+-----+

    # SQL search and delete records from database table 'wp_options' where 'option_name' match 'foo'
    wp db query "DELETE from wp_options where option_id in ($(wp db query "SELECT GROUP_CONCAT(option_id SEPARATOR ',') from wp_options where option_name like '%foo%';" --silent --skip-column-names))"

    # Search for a string and print the result as a table
    $ wp db search https://localhost:8889 --format=table --fields=table,column,match
    +------------+--------------+-----------------------------+
    | table      | column       | match                       |
    +------------+--------------+-----------------------------+
    | wp_options | option_value | https://localhost:8889      |
    | wp_options | option_value | https://localhost:8889      |
    | wp_posts   | guid         | https://localhost:8889/?p=1 |
    | wp_users   | user_url     | https://localhost:8889      |
    +------------+--------------+-----------------------------+

    # Search for a string and get only the IDs (only works for a single table)
    $ wp db search https://localhost:8889 wp_options --format=ids
    1 2

### GLOBAL PARAMETERS

These [global parameters](https://make.wordpress.org/cli/handbook/config/) have the same behavior across all commands and affect how WP-CLI interacts with WordPress.

| **Argument**    | **Description**              |
|:----------------|:-----------------------------|
| `--path=<path>` | Path to the WordPress files. |
| `--url=<url>` | Pretend request came from given URL. In multisite, this argument is how the target site is specified. |
| `--ssh=[<scheme>:][<user>@]<host\|container>[:<port>][<path>]` | Perform operation against a remote server over SSH (or a container using scheme of "docker", "docker-compose", "docker-compose-run", "vagrant"). |
| `--http=<http>` | Perform operation against a remote WordPress installation over HTTP. |
| `--user=<id\|login\|email>` | Set the WordPress user. |
| `--skip-plugins[=<plugins>]` | Skip loading all plugins, or a comma-separated list of plugins. Note: mu-plugins are still loaded. |
| `--skip-themes[=<themes>]` | Skip loading all themes, or a comma-separated list of themes. |
| `--skip-packages` | Skip loading all installed packages. |
| `--require=<path>` | Load PHP file before running the command (may be used more than once). |
| `--exec=<php-code>` | Execute PHP code before running the command (may be used more than once). |
| `--context=<context>` | Load WordPress in a given context. |
| `--[no-]color` | Whether to colorize the output. |
| `--debug[=<group>]` | Show all PHP errors and add verbosity to WP-CLI output. Built-in groups include: bootstrap, commandfactory, and help. |
| `--prompt[=<assoc>]` | Prompt the user to enter values for all command arguments, or a subset specified as comma-separated values. |
| `--quiet` | Suppress informational messages. |

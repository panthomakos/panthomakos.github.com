---
layout: post
title: FULLTEXT Index and SPATIAL Index - MySQL Compatible Rake Tasks
category: Tutorial
tags: [metaprogramming, mysql, rails]
---

Have you ever run a rake task such as `db:test:prepare` or `db:schema:load` only
to be presented with an error about key length?

    Mysql::Error: BLOB/TEXT column 'column-name' used in key specification
      without a key length:
    CREATE INDEX index-name ON table-name (column-name)

Here are some examples I've encountered:

    Mysql::Error: BLOB/TEXT column 'keywords' used in key specification
      without a key length:
    CREATE INDEX fulltext_keywords ON article_search (keywords)

    Mysql::Error: BLOB/TEXT column 'mbr' used in key specification
      without a key length:
    CREATE INDEX spatial_mbr ON client_locations (mbr)


These errors are the result of the [ActiveRecord::SchemaDumper][sd] class not
being able to generate a schema.rb file with proper commands for creating
[FULLTEXT](http://dev.mysql.com/doc/refman/5.5/en/fulltext-search.html) and
[SPATIAL](http://dev.mysql.com/doc/refman/5.5/en/creating-spatial-indexes.html)
MySQL indices.  Fortunately Ruby is a dynamic language, and it's easy to adjust
the way the schema.rb file is generated.

[sd]: http://www.rubydoc.info/github/rails/rails/2109d175f24da725313f/ActiveRecord/SchemaDumper

**NOTE**:  This code assumes your FULLTEXT indices have "fulltext" in their name
and that your SPATIAL indices have "spatial" in their name.  If you don't follow
this convention then you'll need to write your own regular expressions into the
code.

<ol>
  <li>
Create a file called `sd_indexes.rb` in your `config/initializers` directory,
and add the following code.
{% highlight ruby %}
module ActiveRecord
  class SchemaDumper
    def indexes(table, stream)
      if (indexes = @connection.indexes(table)).any?
        add_index_statements = indexes.map do |index|

          if index.name =~ <strong>/fulltext/i</strong>
            "  execute \"CREATE FULLTEXT INDEX #{index.name} ON #{index.table} (#{index.columns.join(',')})\""
          elsif index.name =~ <strong>/spatial/i</strong>
            "  execute \"CREATE SPATIAL INDEX #{index.name} ON #{index.table} (#{index.columns.join(',')})\""
          else
            statment_parts = [('add_index ' + index.table.inspect)]
            statment_parts << index.columns.inspect
            statment_parts < ' + index.name.inspect)
            statment_parts < true' if index.unique

            '  ' + statment_parts.join(', ')
          end
        end

        stream.puts add_index_statements.sort.join("\n")
        stream.puts
      end
    end
  end
end
{% endhighlight %}
This code overrides the basic index creation of the `ActiveRecord::SchemaDumper`
class.  When the schema dumper encounters an index, before issuing the standard
`add_index` command, it checks if the index name matches **/fulltext/i** or
**/spatial/i**.  If there is a match, the schema dumper issues a MySQL
[CREATE INDEX](http://dev.mysql.com/doc/refman/5.5/en/create-index.html) command
to create the FULLTEXT or SPATIAL index manually.
  </li>
  <li>
Generate the schema.rb file so that your changes take effect.
    {% highlight bash %}rake db:schema:dump{% endhighlight %}
  </li>
  <li>
Create your test database from the newly generated schema.rb file.
    {% highlight bash %}rake db:test:prepare{% endhighlight %}
  </li>
</ol>

The added benefit of overriding the `ActiveRecord::SchemaDumper#indexes`
function is that the schema.rb file will be properly generated from now on. That
means that the next time you run a migration you won't have to make any
adjustments to the schema.rb file, and you won't have to re-run the
`db:schema:dump` rake task.

# ServiceNow Query Builder

🟢 This project is production ready.

## About

This library aims to make Encoded Queries easier to read, write, and maintain in script fields on the ServiceNow platform. Encoded queries, especially ones with complex conditions are notoriously difficult to read and comprehend without the aid of the Condition Builder.  This API aims to fix that by offering the following advantages:

**Advantages**
- **Natural language:** A simple, expressive language makes the query easier to read
- **Multi-Line Expressions:** Syntax allows easily spreading the query over multiple lines for better readability
- **Code folding:** Nested expressions make it easy to collapse part of a query or the whole thing so you can focus on just the parts of the code you are interested in
- **100% API Coverage:** Everything you can do with Encoded Queries, you can do with Query Builder. Because the API returns simple strings, you don't have to worry about whether or not an operator or table join is supported.
- **Composable:** Because the API uses pure functions, you can easily extend the API by creating any short hand functions you want in your own scope. No monkey patching required.
- **No Name Collisions:** All the functions are within the Scoped Application namespace which allows keeping the function names simple without risk of name collisions
- **Universal:** Easily use this tool with GlideRecord, GlideQuery, or any other API that accepts encoded query strings.

The ServiceNow Query builder simplifies this by providing a robust API that allows writing Encoded Queries like the one below:

``` js
var _ = x_0505_cc_queryb;
var query = 
    _.matchOneSubquery(
        _.matchAll(
            _.where('active', true),
            _.matchOne(
                _.where('state', '=', 'Open'),
                _.where('state', '=', 'Pending')
            ),
            _.matchOne(
                _.where('category', '=', 'Inquiry'),
                _.where('assigned_to', 'DYNAMIC', '90d1921e5f510100a9ad2572f2b477fe')
            )
        ),
        _.matchAll(
            _.where('active', false)
        )
    );

gs.debug(query);
```

Using a more traditional GlideRecord condition builder, this same query is much more difficult to read, reason through, and edit.  Additionally, the code takes up a large footprint and can not be collapsed:

```js
var gr = new GlideRecord('incident');
gr.addQuery('active', true);

var stateCond = gr.addQuery('state', 'Open');
stateCond.addOrCondition('state', 'Pending');

var categoryCond = gr.addQuery('category', 'Inquiry');
categoryCond.addOrCondition('assigned_to', 'DYNAMIC', '90d1921e5f510100a9ad2572f2b477fe');

gr.addEncodedQuery('^NQactive=false');

gs.debug(gr.getEncodedQuery());
```

Similarly, representing the entire query as an Encoded Query is possible.  This makes it easier to edit since a new Encoded Query can simply be copied from a Condition Builder and keeps it compact but it does little to address readabilty:

```js
var query = 'active=true^state=Open^ORstate=Pending^category=Inquiry^ORassigned_toDYNAMIC90d1921e5f510100a9ad2572f2b477fe^NQactive=false';
gs.debug(query);
```

A slightly improved Encoded Query approach spreads it over multiple lines for readability but once again the query takes up a large footprint that can not be collapsed:

```js
var query = 'active=true' 
+ '^state=Open'
+ '^ORstate=Pending'
+ '^category=Inquiry' 
+ '^ORassigned_toDYNAMIC90d1921e5f510100a9ad2572f2b477fe'
+ '^NQactive=false';
gs.debug(query);
```

GlideQuery offers a more compelling API and many desireable query execution features but suffers a number of limitations in query building including lack of support for all 3 levels of query nesting (NQ, AND, OR), inability to parse some operators like 'DYNAMIC', lack of support for more advanced queries.  It also still suffers from a large footprint with no code folding for complex queries.  The following example will not actually work but I believe approximates how the API might be able to handle these queries in the future:

```js
// Currently throws an error
var gq = new GlideQuery('incident')
.where(new GlideQuery('incident')
    .where('active', true)
    .where(new GlideQuery('incident')
        .where('state', 'Open')
        .orWhere('state', 'Pending')
    )
    .where(new GlideQuery('incident')
        .where('category', 'Inquiry')
        .orWhere('assigned_to', 'DYNAMIC', '90d1921e5f510100a9ad2572f2b477fe')
    )
)
.orWhere(new GlideQuery('incident')
    .where('active', false)
)

gs.debug(gq.getGlideRecord().getEncodedQuery());
```

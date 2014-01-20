##Retrieving comments from Google Spreadsheet

In one of our project, we are using Google Spreadsheet as a major data source. We utilized Roo::Google in [Roo Gem](https://github.com/Empact/roo) to access the spreadsheet. Roo actually uses [Google Drive Ruby Gem](https://github.com/gimite/google-drive-ruby) to communicate with [Google Spreadsheet API v3](https://developers.google.com/google-apps/spreadsheets/#working_with_cell-based_feeds).

Everything works pretty well until recently, when we have to access the comments on individual cells in the spreadsheet.
![cell comments](https://raw2.github.com/pyu10055/reading_google_spreadsheet_comments/master/comments.png)

Cell comments feature is not exposed by [Google Spreadsheet API v3](https://developers.google.com/google-apps/spreadsheets/#working_with_cell-based_feeds),
as a result, we can not retrieve those comments through Roo::Google in [Roo Gem](https://github.com/Empact/roo).

We googled around a little bit, people are suggesting using cell notes instead of comments since it is available when you export the spreadsheet to html format.
Cell notes is a feature google provided before cell comments, but it lacks commentator information and other features cell comments provides.
However, the html code Google generated from the spreadsheet actually is malformatted and very hard to link the note back to the cell by scraping the html.
Following is an example html export, you can notice most table cells are not closed.

```
<table dir='ltr' border=0 cellpadding=0 cellspacing=0 class='tblGenFixed' id='tblMain'>
  <tr class='rShim'>
    <td class='rShim' style='width:0;'>
    <td class='rShim' style='width:120px;'>
  <tr dir='ltr'>
    <td class=hd><p style='height:16px;'>.</td>
    <td  class='s0'>1
  </tr>
  <tr dir='ltr'>
    <td class=hd><p style='height:16px;'>.</td>
    <td  class='s1'>2%
    <span class='annotation'> [1]</span>
  </tr>
</table>
<div class='annotations'>[1] <span>cell note</span><br></div>
```

We decide to look further. We export the spreadsheet into excelx format. XLSX file is actually just a zip file, we were positively surprised when we unzip that file:

![xlsx content](https://raw2.github.com/pyu10055/reading_google_spreadsheet_comments/master/excelx.png)

It has a file called comments1.xml, and its content is exact what we need.

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<comments xmlns="http://schemas.openxmlformats.org/spreadsheetml/2006/main">
  <authors><author></author></authors>
  <commentList>
    <comment ref="A2" authorId="0"><text><t xml:space="preserve">cell note</t></text></comment>
    <comment ref="A1" authorId="0"><text><t xml:space="preserve">testing comments	-Ping Yu</t></text></comment>
  </commentList>
</comments>
```

But when we open this xlsx file with Roo::Excelx reader in [Roo Gem](https://github.com/Empact/roo), the reader cannot find any comments.
It is caused by a slightly different structure between what Google generates and what Roo expects.
We submitted a [PR  here](https://github.com/Empact/roo/pull/95), you can use our [repo](https://github.com/intridea/roo) before the PR is merged.

Here is the code snippet for exporting the spreadsheet and retrieve the comment:

```
  drive = GoogleDrive.login("username@gmail.com", "mypassword")

  path =
    Dir::Tmpname.make_tmpname("#{Rails.root}/tmp/raw", nil) + ".xlsx"

  # example google spreadsheet url
  # https://docs.google.com/spreadsheet/ccc?key=pz7XtlQC-PYx-jrVMJErTcg
  drive.spreadsheet_by_key("pz7XtlQC-PYx-jrVMJErTcg").export_as_file(path, "xls")

  Roo::Excelx.new(path, comment_xpath: './xmlns:text/xmlns:t').comments
  #[[129, 60, "New comment"], [156, 5, "comments\n\t-Ping Yu"]]

```

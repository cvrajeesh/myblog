title: "JQueryUI Datepicker in ASP.Net MVC"
date: 2010-02-28 12:10:57
tags:
---

Datepicker is nice and cool plugin for displaying the calendar with ease. It is very easy to use JQuery plugin, it comes as part of JQueryUI library, so if you want to use this – first download JQueryUI from http://jqueryui.com/download and also download JQuery(http://jquery.com/download/) if you haven’t done yet.

Assume you have a form like one below

```
<% using(Html.BeginForm()){%>
    <fieldset>
    <legend>Event Information</legend>
      <p>
        <label for="EventName">Event Name:</label>
        <%= Html.TextBox("EventName")%>
      </p>
      <p>
        <label for="StartDate">Start Date:</label>
        <%= Html.TextBox("StartDate")%>
      </p>
      <p>
        <label for="EndDate">End Date:</label>
        <%= Html.TextBox("EndDate")%>
      </p>
      <p>
        <input type="submit" value="Save" />
      </p>
  </fieldset>
<% }%>
```

and you want to attach datepicker to `StartDate` and `EndDate` input fields, what you needs to do is call the `datepicker` function on the these input field selector like below.

```js
$(document).ready(function() {
  $('#StartDate').datepicker();
  $('#EndDate').datepicker();
});
```

This works fine as we expected :)

###Difference in Date format patterns

Consider a scenario where your MVC application supports localization, then the selected date displayed in the input fields also should display the in the same date format of the current culture(This format could be custom one or default one).

This leads to you another issue – Datepicker plugin given by the JQueryUI supports different date formats, but it is different from one that is available in .NET. For e.g. in order to display a long day name (“Thursday”) .NET uses **dddd*** its equivalent in Datepicker is **DD**.

In order to solve this disparity between the .NET world and Datepicker world, I have created a html helper function, which could generate the Datepicker format from a .Net date format.

```cs
/// <summary>
/// JQuery UI DatePicker helper.
/// </summary>
public static class JQueryUIDatePickerHelper
{
    /// <summary>
    /// Converts the .net supported date format current culture format into JQuery Datepicker format.
    /// </summary>
    /// <param name="html">HtmlHelper object.</param>
    /// <returns>Format string that supported in JQuery Datepicker.</returns>
    public static string ConvertDateFormat(this HtmlHelper html)
    {
        return ConvertDateFormat(html, Thread.CurrentThread.CurrentCulture.DateTimeFormat.ShortDatePattern);
    }

    /// <summary>
    /// Converts the .net supported date format current culture format into JQuery Datepicker format.
    /// </summary>
    /// <param name="html">HtmlHelper object.</param>
    /// <param name="format">Date format supported by .NET.</param>
    /// <returns>Format string that supported in JQuery Datepicker.</returns>
    public static string ConvertDateFormat(this HtmlHelper html, string format)
    {
        /*
         *  Date used in this comment : 5th - Nov - 2009 (Thursday)
         *
         *  .NET    JQueryUI        Output      Comment
         *  --------------------------------------------------------------
         *  d       d               5           day of month(No leading zero)
         *  dd      dd              05          day of month(two digit)
         *  ddd     D               Thu         day short name
         *  dddd    DD              Thursday    day long name
         *  M       m               11          month of year(No leading zero)
         *  MM      mm              11          month of year(two digit)
         *  MMM     M               Nov         month name short
         *  MMMM    MM              November    month name long.
         *  yy      y               09          Year(two digit)
         *  yyyy    yy              2009        Year(four digit)             *
         */

        string currentFormat = format;

        // Convert the date
        currentFormat = currentFormat.Replace("dddd", "DD");
        currentFormat = currentFormat.Replace("ddd", "D");

        // Convert month
        if (currentFormat.Contains("MMMM"))
        {
            currentFormat = currentFormat.Replace("MMMM", "MM");
        }
        else if (currentFormat.Contains("MMM"))
        {
            currentFormat = currentFormat.Replace("MMM", "M");
        }
        else if (currentFormat.Contains("MM"))
        {
            currentFormat = currentFormat.Replace("MM", "mm");
        }
        else
        {
            currentFormat = currentFormat.Replace("M", "m");
        }

        // Convert year
        currentFormat = currentFormat.Contains("yyyy") ? currentFormat.Replace("yyyy", "yy") : currentFormat.Replace("yy", "y");

        return currentFormat;
    }
}
```

So how we could make use this helper method, just replace the datepicker initialization code we have written earlier with this

```js
$(document).ready(function() {
  $('#StartDate').datepicker({ dateFormat: '<%= Html.ConvertDateFormat() %>' });
  $('#EndDate').datepicker({ dateFormat: '<%= Html.ConvertDateFormat() %>' });
});
```

Hope this helps.

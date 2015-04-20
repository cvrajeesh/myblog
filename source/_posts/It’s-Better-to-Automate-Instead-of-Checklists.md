title: "It’s Better to Automate, Instead of Checklists"
date: 2010-09-20 12:46:38
tags:
---

In my day to day activities I have seen many checklists like

1. [Code review](http://en.wikipedia.org/wiki/Code_review) checklist
2. Source control check-in checklist
2. Developer checklist

All these are good because it helps to reduce failures but does everyone follow these all the time?. Sometimes I (or any developer) forgot to go through the checklist due to many reasons like time constraints, lack of concentration etc… and I don’t think we should blame anyone for missing this because - *We all are humans and we tends to forget*.

Only way we could reduce these mistakes is to automate!!! wherever possible. In my current project, all the aspx page should have direction(dir) attribute in the html tag as part of the localization work. As usual an email with  checklist for localizing an aspx page was sent to all the developers, out of that one item was to include `dir` attribute whenever they add new aspx file.

Everybody followed this in the initial stages but later we started forgetting about this requirement, which caused extra hours of effort to fix it in all the pages.

It could have been avoided if we had a automated process which verifies this. In order to automate, one way is to write a custom MSBuild task which could verify whether a aspx file has `dir` attribute, if it doesn’t fails build (this whole idea came from http://blogs.msdn.com/b/simonince/archive/2009/07/10/enforcing-unobtrusive-javascript.aspx). If you want to learn about writing a custom MSBuild task, I suggest http://msdn.microsoft.com/en-us/library/t9883dzc.aspx.

So here is the  code which creates this custom MS Build task

```cs
using System.Linq;
using HtmlAgilityPack;
using Microsoft.Build.Framework;
using Microsoft.Build.Utilities;

namespace StaticFileAnalysisTask
{
    /// <summary>
    /// Custom MSBuild task which analyse the HTML code validation.
    /// </summary>
    public class HTMLCodingStandard : Task
    {
        #region Private fields

        /// <summary>
        /// Variable that holds the status of this task execution.
        /// </summary>
        private bool _success;

        #endregion

        #region Public properties

        /// <summary>
        /// Gets or sets the source file list.
        /// </summary>
        /// <value>The source file list.</value>
        [Required]
        public ITaskItem[] SourceFiles { get; set; }

        /// <summary>
        /// Gets or sets the list of files to be excluded.
        /// </summary>
        /// <value>The excluded files.</value>
        public ITaskItem[] ExcludeFiles { get; set; }

        #endregion

        /// <summary>
        /// Executes this task.
        /// </summary>
        /// <returns><c>true</c> if task executed successfully; Otherwise, <c>false</c>.</returns>
        public override bool Execute()
        {
            this._success = true;

            foreach (ITaskItem current in SourceFiles)
            {
                // If the items is in the exluded list, then skip
                if(this.IsInExlcudedList(current))
                {
                    continue;
                }

                string path = current.ItemSpec;
                if (path.EndsWith(".aspx"))
                {
                    this.ValidateFile(path);
                }
            }
            return this._success;
        }

        /// <summary>
        /// Method that is responsible for validating the file.
        /// </summary>
        /// <param name="path">The full path to the file.</param>
        private void ValidateFile(string path)
        {
            HtmlDocument htmlDocument = new HtmlDocument();
            htmlDocument.Load(path);
            HtmlNode node = htmlDocument.DocumentNode.SelectSingleNode("//html");
            if(node != null)
            {
                if(!node.Attributes.Contains("dir"))
                {
                    this.BuildEngine.LogErrorEvent(new BuildErrorEventArgs(
                        "Invalid HTML coding standard",
                        "SFAT-HTML-1",
                        path,
                        node.Line,
                        node.LinePosition,
                        node.LinePosition,
                        node.LinePosition + node.OuterHtml.Length,
                        "SFAT-HTML-1: Direction(dir) tag is missing",
                        null,
                        "HTMLCodingStandardTask"
                        ));
                    this._success = false;
                }

            }
        }

        /// <summary>
        /// Determines whether an item is in the excluded list.
        /// </summary>
        /// <param name="taskItem">The task item which needs to checked.</param>
        /// <returns>
        ///     <c>true</c> if item is in exlcuded list; otherwise, <c>false</c>.
        /// </returns>
        private bool IsInExlcudedList(ITaskItem taskItem)
        {
            if(this.ExcludeFiles == null)
            {
                return false;
            }

            return this.ExcludeFiles.Any(x => x.ItemSpec == taskItem.ItemSpec);
        }
    }
}
```

For including this in the your web project, open the project file in a text editor and add the below lines

```xml
<UsingTask AssemblyFile="..\output\StaticFileAnalysisTask.dll" TaskName="HTMLCodingStandard" />
  <Target Name="BeforeBuild">
    <HTMLCodingStandard SourceFiles="@(Content)" />
  </Target> If you want to exclude any files from this verification process, then define an ItemGroup
<ItemGroup>
    <ExcludedContents Include="WebForm2.aspx" />
  </ItemGroup> and add that to the build action like below
 <UsingTask AssemblyFile="..\output\StaticFileAnalysisTask.dll" TaskName="HTMLCodingStandard" />
  <Target Name="BeforeBuild">
    <HTMLCodingStandard SourceFiles="@(Content)" ExcludeFiles="@(ExcludedContents)" />
  </Target>
```
Hope this helps you.

> Download code sample from [here](http://rajeesh.cdn.rhyble.com/download/TestWebApplication.zip)

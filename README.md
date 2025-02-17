# Evo.Blazor.Splitters

A .NET 6.0 Blazor Splitter Control that is almost entirely written in C#.  

# How to Use

For right now, to use the splitter, download the Evo.Blazor and Evo.Blazor.Splitters libraries, and compile them or add them your solution directly.  A package on github is in the works.  If you download the entire repository, the solution will contain the examples on how to use the splitter.   

```
<EvoSplitter GutterBarClass="splitterGutterBar" Orientation="SplitOrientation.Horizontal">
    <EvoSplitterPanes>
        <EvoSplitterPane Class="splitterPane" Style="user-select: none;" MinimumSizeInPixels="28">
            <div class="splitterPane" style="user-select: none;" >inner content 1</div>
        </EvoSplitterPane>
        <EvoSplitterPane MinimumSizeInPixels="26">
            <div class="splitterPane" style="user-select: none;" >inner content 2</div>
        </EvoSplitterPane>
    </EvoSplitterPanes>
    
</EvoSplitter>
```

# Project

The rest of this document is dedicated to provide information about how the project came together and explain how the control was developed.  The work was not trivial and required a good amount of research.  While other developers could probably have done this quicker, a lot of other developers will probably struggle with these concepts, as much, or more.  To aide them, and to give back to the community, the project notes are below.

# Purpose 

1) Create a splitter control that can be used to seperate a control area from a canvas
1) Learn what is needed to create a decently complex custom control in Blazor using .NET 5


## Create a Splitter Control

This project came up as a neccessity to build out another larger work, and thus was not random.  Thus the primary purpose of this project was to learn how to create a slidder control where on of the divs can contain a canvas.  In addition, the canvas's dimensions needed to be kept up to date as the parent div changed its size and dimensions.   Several projects were found, but nothing up to date was found that could fit what was needed.  Thus, decided to create a new one from scratch, though with some help here and there from some existing open source libraries.  

## Learn .NET 5 Custom Controls Development

Been around a the block a few times with Blazor, but have not had the chance yet to actaully build out a custom control with .NET 6 with any decent amount of complexity.  Blazor has been changing a lot over the years, and this was a great refresher course.  After researching what is all involved in making a slider control work, this project appeared to be a good opportunity to learn about all the complex nuances of building out a control from scratch.  In the end, it was.  The effort required using paramaters, child controls, cascading parameters, the javascript interop, writing custom rendering routines, learning about all the mouse events, investigating drag and drop (though ulitmaely, for now, decided not to use), and how to utilize the resizeObserver pattern.  In addition, created a message bus to enable controls to communicate asynchrounously without havig to lookups each time an event was fired.  

# Goals

- Avoid as much javascript as possible - The goal is to create a native solution if at all possible and not have to mess with a javascript implementation.  Split.js already does this work well, and the purpose of this project is to learn how to create a native control that can do the same.  
- Inject as litle as possible into the DOM when implementing the control.  One of the goals of Splitter.js was to keep the footprint low.  It was decided that this project would have the same goal.  While gutter/splitter bars will need to be added to seperate the div panes from one another, not having to create wrapper DIV elements is another if they already exists.  There are two ways of approaching this. The first is to use javascript to edit what is already there; this was frowned upon.  The second is to use an HTML agility pack and parse the HTML and rebuild the outer divs within the library; the later was ultimately chosen as it eliminates additional javascript from the equation, which is the first goal.
- Create a performant solution.  While the solution can probably still be tuned a bit more, the one created here results in smooth slidding and minimizes updates to any child controls.  

# Implementation

The following are some notes on how the project was implemented and how the projects was put together.  Given there was a lot of learning involved, the plan was to take the everything one step at a time and not try to create a perfect solution the first go around.  Instead, the plan was to get it working then refine as neccessary to come up with a performant slidder solution that met the goals above.  

## Get Two Divs to Show Up

The first step of the project was to get two divs to show up on the screen.  

### Format the Controls Statically

The Make the bar the right height, and color, give the panes a background color, use a calc expression to handle height.

### Make them Appear Dynamically / Setup Child Controls

## Build the Gutter / Splitter Bar

## Make the Slider Work

## Setup Services

## Made Services Injectable

## Setup Pane Registration

## Add Logic to Determine If Pane is Last



## Make Formating Appear Dynamically
The next step is to create a div that appears after every pane div.  

## Use Cascading Values or Models to Propigate Settings

## Rendering



### Reuse Existing Divs

This project shares some of the same goals that splitter.js has, including the goal of keeping the dom overhead as light possible and not to impose elements onto to the developer if they are not needed.  To this end, the evo splitter pane needs to render two different ways depending upon whether child content's root element is a div element.  If the child content root is a div, then the code will replace the root div, and only render the content within the childs content root element.  If the child content has more than one root, or the root element is not a div, then the content will be wrapped in a div.  

Below is the first evolution of this code that demonstrates the basic technique.  It might be possible to go back to a standard razor page, but that might take some experimentation.  The only gotcah is dynamically filling in the attributes.  [Initial research points that this might be unlikely.](https://stackoverflow.com/questions/64196493/in-blazor-how-can-i-dynamically-change-an-html-tag).
```
public partial class EvoSplitterPane: EvoSplitterPaneBase
    {
        private bool _TryUseExistingDiv = true;
        private bool _UseExistingDiv = false;
        private bool _FirstRender = true;
        private string _InnerDivHtml = null;
        private Dictionary<string, object> _outerDivAttributes;
        private string _OrignalStyleValue;

        // One strategy could be to read the div and its attributes

        protected override void BuildRenderTree(RenderTreeBuilder builder)
        {
#pragma warning disable BL0006

            if (_FirstRender && _TryUseExistingDiv)
            {
                _FirstRender = false;

                AttemptToUseExistingDiv();
            }

            if (_UseExistingDiv)
            {
                builder.OpenElement(0, "div");

                if (string.IsNullOrWhiteSpace(_OrignalStyleValue))
                {
                    builder.AddAttribute(1, "style", DynamicStyle);
                }
                else
                {
                    //
                    builder.AddAttribute(1, "style", $"{_OrignalStyleValue};{DynamicStyle}");
                }

                builder.AddMultipleAttributes(2, _outerDivAttributes);

                builder.AddAttribute(3, "b-9gda3aicu2");

                builder.AddContent(4, _InnerDivHtml);
                builder.CloseElement();
                if (!this.IsLastPane())
                {
                    TypeInference.CreateCascadingValue_0(builder, 5, 6, this, 7, (__builder2) =>
                    {
                        __builder2.OpenComponent<EvoGutterBar>(8);
                        __builder2.CloseComponent();
                    });
                }
            }
            else
            {
                builder.OpenElement(0, "div");
                builder.AddAttribute(1, "style", DynamicStyle);
                builder.AddAttribute(2, "b-9gda3aicu2");
                builder.AddContent(3, ChildContent);
                builder.CloseElement();

                if (!this.IsLastPane())
                {
                    TypeInference.CreateCascadingValue_0(builder, 4, 5, this, 6, (__builder2) =>
                    {
                        __builder2.OpenComponent<Evo.Blazor.EvoGutterBar>(7);
                        __builder2.CloseComponent();
                    });
                }
            }
        }

        public void AttemptToUseExistingDiv()
        {
            if (!TryGetMarkupFrame(ChildContent, out var markupFrame))
            {
                Console.WriteLine("Could not get mark up frame.");
                _UseExistingDiv = false;
                return;
            }

            if (!TryGetCleanMarkupContent(markupFrame, out var markupContent))
            {
                Console.WriteLine("Could not clean content");
                _UseExistingDiv = false;
                return;
            }

            var htmlDoc = new HtmlDocument();

            htmlDoc.LoadHtml(markupContent);

            var documentNode = htmlDoc.DocumentNode;

            if (documentNode.ChildNodes.Count > 1)
            {
                Console.WriteLine("Not a single root node in child content");
                _UseExistingDiv = false;
                return;
            }

            var childNode = documentNode.ChildNodes[0];

            if (childNode.NodeType != HtmlNodeType.Element)
            {
                Console.WriteLine("Single node is not an element.");
                _UseExistingDiv = false;
                return;
            }

            Console.WriteLine(childNode.Name);

            if (childNode.Name != "div")
            {
                Console.WriteLine("Node is not a 'div' node.");
                _UseExistingDiv = false;
                return;
            }

            _InnerDivHtml = childNode.InnerHtml;

            _outerDivAttributes = new Dictionary<string, object>();

            foreach(var attribute in childNode.Attributes)
            {
                if (attribute?.OriginalName?.ToLower() == "style")
                {
                    _OrignalStyleValue = attribute.Value;
                }
                else
                {
                    _outerDivAttributes.Add(attribute.OriginalName, attribute.Value);
                }
            }

            _UseExistingDiv = true;
        }

        private bool TryGetCleanMarkupContent(RenderTreeFrame markupFrame, out string cleanContent)
        {
            if (markupFrame.FrameType != RenderTreeFrameType.Markup)
            {
                cleanContent = null;
                return false;
            }

            var content = markupFrame.MarkupContent;

            if (content == null)
            {
                cleanContent = null;
                return false;
            }

            cleanContent = content?.Trim();

            return true;
        }

        private bool TryGetMarkupFrame(RenderFragment fragment, out RenderTreeFrame markupFrame)
        {
            var builder = new RenderTreeBuilder();

            builder.AddContent(0, fragment);

            var frames = builder.GetFrames();

            foreach (var frame in frames.Array)
            {
                if (frame.FrameType == RenderTreeFrameType.Markup)
                {
                    markupFrame = frame;
                    return true;
                }
            }

            markupFrame = default;
            return false;
        }

        protected override Task OnAfterRenderAsync(bool firstRender)
        {
            return base.OnAfterRenderAsync(firstRender);
        }

        internal static class TypeInference
        {
            public static void CreateCascadingValue_0<TValue>(RenderTreeBuilder __builder, int seq, int __seq0, TValue __arg0, int __seq1, global::Microsoft.AspNetCore.Components.RenderFragment __arg1)
            {
                __builder.OpenComponent<global::Microsoft.AspNetCore.Components.CascadingValue<TValue>>(seq);
                __builder.AddAttribute(__seq0, "Value", __arg0);
                __builder.AddAttribute(__seq1, "ChildContent", __arg1);
                __builder.CloseComponent();
            }
        }
    }
```

## Consolidate the Measurement Process

The initial design was setup where the control would be measured after reach render was complete.  After implementing an [ResizeObserver](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver/ResizeObserver) pattern to enable detection of the div being resized for any reason, it was thought it may be better to receive async updatesoly from the javascript and not call the RefreshMeasurementsAsync after each refresh.  Numerically, this worked, but there was an noticable flicker on the screen when the splitter div pane contained a child canvas.  Thus both are used to update the measurements.

# References

[Column Resizable](https://github.com/alvaro-prieto/colResizable/blob/master/colResizable-1.6.js)

[Splitter.js Reference Implementation](https://github.com/nathancahill/split/blob/a206479d81a6b2a69ba5742f57fd378d997fd8e7/packages/splitjs/src/split.js#L42)

[Modernizer Calc Function](https://stackoverflow.com/questions/16625140/js-feature-detection-to-detect-the-usage-of-webkit-calc-over-calc/16625167#16625167)

[Radzen Blazor (MIT Licensed)](https://github.com/radzenhq/radzen-blazor/tree/master/Radzen.Blazor) - Specifically useful was the [tab control implementation]()

[Working Example using Splitter.js](http://jsfiddle.net/aguiguy/y2x7fdnu/)

[BlazorSplit](https://github.com/BlazorComponents/BlazorSplit) - An older implementation of a blazor split control that uses javascript.

[Investigating Drap and Drop with Blazor](https://chrissainty.com/investigating-drag-and-drop-with-blazor/)

[Browser Interop](https://github.com/RemiBou/BrowserInterop/tree/master/src/BrowserInterop)

[Computing Css Scope - Source Code](https://github.com/dotnet/aspnetcore/blob/a190fd34854542266956b1af980c19afacb95feb/src/Razor/Microsoft.NET.Sdk.Razor/src/ComputeCssScope.cs#L43)

[Css Isolation in Blazor](https://www.meziantou.net/css-isolation-in-blazor.htm)

Splitter Examples - [Example 1](http://jsfiddle.net/aguiguy/y2x7fdnu/) and [Example 2](https://jsfiddle.net/8qmbywr1/)

[Blazored](https://github.com/Blazored)

[Blazor Source Code](https://github.com/dotnet/aspnetcore/tree/c925f99cddac0df90ed0bc4a07ecda6b054a0b02/src/Components/Components/src)

[Lupusa87/BlazorSplitter](https://github.com/Lupusa87/BlazorSplitter/tree/master/BlazorSplitterComponent)

https://coder-coder.com/display-divs-side-by-side/

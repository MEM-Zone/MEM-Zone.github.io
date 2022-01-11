# Built-in Shortcodes Tests


<!--more-->

## figure

{{< figure src="https://octodex.github.com/images/minion.png" title="octodex" >}}

## gist

{{< gist spf13 7896402 >}}

## highlight

{{< highlight html >}}
<section id="main">
    <div>
        <h1 id="title">{{ .Title }}</h1>
        {{ range .Pages }}
            {{ .Render "summary"}}
        {{ end }}
    </div>
</section>
{{< /highlight >}}

## param

{{< param description >}}

## ref and relref

[ref]({{< ref "about/index.en.md" >}})

[relref]({{< relref "../../../about/index.en.md" >}})

## tweet

{{< tweet 877500564405444608 >}}

## vimeo

{{< vimeo 146022717 >}}

## youtube

{{< youtube w7Ft2ymGmfc >}}


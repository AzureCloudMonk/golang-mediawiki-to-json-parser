# golang-mediawiki-to-html-parser
Golang MediaWiki to HTML Parser

Here's the entire previous explanation in GitHub markdown format:

---

# MediaWiki Parser to JSON in Go

This Go program parses MediaWiki-like syntax into JSON. This parser will handle common MediaWiki elements such as headings, bold, italics, internal links, and external links. Each parsed element will be represented in JSON, allowing for structured representation of the content.

The JSON output structure might look like this:

```json
{
  "title": "Example Page",
  "content": [
    {"type": "heading", "level": 1, "text": "Welcome to the Wiki"},
    {"type": "paragraph", "text": "This is a simple page about Golang."},
    {"type": "bold", "text": "Golang Page"},
    {"type": "link", "title": "Golang", "url": "/page/Golang"}
  ]
}
```

## Code

Here's the single-file Go program:

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "regexp"
    "strings"
)

// ContentItem represents a single parsed item from the MediaWiki text.
type ContentItem struct {
    Type  string `json:"type"`
    Level int    `json:"level,omitempty"` // For headings only
    Text  string `json:"text,omitempty"`  // For headings, bold, italic, and paragraph
    Title string `json:"title,omitempty"` // For internal links
    URL   string `json:"url,omitempty"`   // For external links
}

// ParseMediaWiki parses MediaWiki-like syntax into JSON-structured content.
func ParseMediaWiki(text string) ([]ContentItem, error) {
    var content []ContentItem

    lines := strings.Split(text, "\n")
    for _, line := range lines {
        line = strings.TrimSpace(line)
        if line == "" {
            continue
        }

        switch {
        case strings.HasPrefix(line, "="): // Headings
            content = append(content, parseHeading(line))
        case strings.HasPrefix(line, "'''"): // Bold
            content = append(content, ContentItem{Type: "bold", Text: parseBold(line)})
        case strings.HasPrefix(line, "''"): // Italic
            content = append(content, ContentItem{Type: "italic", Text: parseItalic(line)})
        case strings.HasPrefix(line, "[["): // Internal Links
            content = append(content, parseInternalLink(line))
        case strings.HasPrefix(line, "["): // External Links
            content = append(content, parseExternalLink(line))
        default: // Regular text (paragraph)
            content = append(content, ContentItem{Type: "paragraph", Text: line})
        }
    }

    return content, nil
}

// parseHeading parses MediaWiki-style headings, e.g., `= Heading =`
func parseHeading(line string) ContentItem {
    headingRegex := regexp.MustCompile(`^(=+)\s*(.*?)\s*\1$`)
    matches := headingRegex.FindStringSubmatch(line)
    level := len(matches[1]) // Number of `=` symbols represents heading level
    return ContentItem{Type: "heading", Level: level, Text: matches[2]}
}

// parseBold parses `'''bold'''` text
func parseBold(line string) string {
    boldRegex := regexp.MustCompile(`'''(.*?)'''`)
    return boldRegex.ReplaceAllString(line, "$1")
}

// parseItalic parses `''italic''` text
func parseItalic(line string) string {
    italicRegex := regexp.MustCompile(`''(.*?)''`)
    return italicRegex.ReplaceAllString(line, "$1")
}

// parseInternalLink parses `[[PageName]]` links to internal pages
func parseInternalLink(line string) ContentItem {
    internalLinkRegex := regexp.MustCompile(`\[\[([^\]]+?)\]\]`)
    matches := internalLinkRegex.FindStringSubmatch(line)
    title := matches[1]
    return ContentItem{Type: "link", Title: title, URL: fmt.Sprintf("/page/%s", title)}
}

// parseExternalLink parses `[http://example.com]` links
func parseExternalLink(line string) ContentItem {
    externalLinkRegex := regexp.MustCompile(`\[(http[^\s]+)\]`)
    matches := externalLinkRegex.FindStringSubmatch(line)
    url := matches[1]
    return ContentItem{Type: "link", URL: url, Title: url}
}

func main() {
    // Example MediaWiki content
    content := `= Welcome to the Wiki =
This is a simple page about ''Golang''. Visit the '''Golang Page''' by clicking [[Golang]].
To learn more, visit [https://golang.org].`

    // Parse the content to JSON structure
    parsedContent, err := ParseMediaWiki(content)
    if err != nil {
        log.Fatal("Error parsing content:", err)
    }

    // Convert parsed content to JSON
    jsonOutput, err := json.MarshalIndent(parsedContent, "", "  ")
    if err != nil {
        log.Fatal("Error generating JSON:", err)
    }

    // Display JSON output
    fmt.Println(string(jsonOutput))
}
```

## Explanation of Each Part

### Structs
- `ContentItem`: Represents a parsed unit from the MediaWiki content, with different fields (like `Text`, `Level`, `Title`, and `URL`) depending on the item type.

### Parsing Functions
- **`parseHeading`**: Matches `= Heading =` syntax, counting the `=` signs to determine the level.
- **`parseBold` and `parseItalic`**: Extract the inner content from bold and italic syntax.
- **`parseInternalLink`**: Matches `[[PageName]]` links to internal wiki pages and formats them as structured data with a `Title` and a URL.
- **`parseExternalLink`**: Matches `[http://example.com]` links to external URLs.

### Main Program
- Parses the example content into JSON and outputs the structured JSON representation.

## Running the Program

To run the program, save it to a file (e.g., `mediawiki_parser.go`), and then run:

```bash
go run mediawiki_parser.go
```

This code is designed to handle basic MediaWiki syntax parsing and output it in JSON format for easy processing and display in various applications.




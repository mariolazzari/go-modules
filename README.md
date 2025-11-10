# Go modules

## Getting started

### Introduction

A Go module is a container for different Go packages

- Name and version
- Published on Github
- Introduced in Go 1.11
- No global workspace
- Flexible code structure
- Improve dependency management
- Go tools
- Reproducible builds

### Creating new module

```sh
mkdir emoserv
cd emoserv
go mod init github/mariolazzari/emoserv
```

### Module structure

- Root dir
- No content limitation
- Mix of package and non package directories
- Structure must fit your needs
- Plan for growth
- Avoid deeply nested structure

### Add packages and source code

```go
package search

import (
	"slices"
	"strings"
)

// Params used to specify search parameters
type Params struct {
	Include []string // Include slice of strings to include in search
	Exclude []string // Exclude slice of strings to exclude in search
}

// ByDescription searches emojis using the params
func ByDescription(params Params) (result []Emoji) {

	for _, emo := range emojis {
		if shouldExclude(emo, params.Exclude) {
			continue
		}

		for _, include := range params.Include {
			include = strings.ToLower(include)
			if strings.Contains(emo.Label, include) || slices.Contains(emo.Tags, include) {
				result = append(result, emo)
			}
		}
	}

	return
}

// version 0.1.0
// func Like(emoji string) (result []emoji) {...}

// shouldExclude checks emoji tags and labels for exclusions
func shouldExclude(emo Emoji, excludes []string) bool {
	for _, exclude := range excludes {
		exclude = strings.ToLower(exclude)
		if strings.Contains(emo.Label, exclude) || slices.Contains(emo.Tags, exclude) {
			return true
		}
	}

	return false
}
```

### Testing packages

```go
package search

import (
	"slices"
	"testing"
)

func TestByDesc(t *testing.T) {
	tests := map[string]struct {
		params Params
		want   []string
	}{
		"fruit": {
			Params{Include: []string{"fruit"}},
			[]string{"🍇", "🍈", "🍉", "🍊", "🍋"},
		},
		"cat": {
			Params{Include: []string{"cat"}},
			[]string{"🐱"},
		},
		"animal faces": {
			Params{Include: []string{"face"}, Exclude: []string{"smile", "laugh", "grin", "upside-down"}},
			[]string{"🐵", "🐶", "🐱", "🐯", "🦊"},
		},
	}

	for name, test := range tests {
		t.Run(name, func(t *testing.T) {
			result := ByDescription(test.params)
			if len(result) != len(test.want) {
				t.Errorf("Search ByDescription: want %d emojis, got %d", len(test.want), len(result))
			}

			for _, emoji := range result {
				if !slices.Contains(test.want, emoji.Emoji) {
					t.Errorf("Search ByDescription: expected %s to be in %#v", emoji, test.want)
				}
			}
		})
	}
}
```

```sh
cd serach
go test .
```

### Documenting module

doc.go

```go
// Package search provides functions to search for and retrieve emojis
package search
```

```sh
go doc -all ./search
```

## Publishing module

### Exported names

```sh
go list ./search
```

### Reusing package

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"

	"lil/emoserv/search"
)

func main() {
	http.HandleFunc("/search", func(w http.ResponseWriter, r *http.Request) {
		var params search.Params
		if err := json.NewDecoder(r.Body).Decode(&params); err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
			return
		}
		log.Printf("Got request: %#v", params)

		// write result
		w.Header().Set("Content-Type", "application/json")
		result := search.ByDescription(params)
		if err := json.NewEncoder(w).Encode(&result); err != nil {
			w.WriteHeader(http.StatusInternalServerError)
			return
		}
	})

	log.Print("Listening on port 8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### Multiple modules

```sh
go mod init lil/emoserv
```

### Reusing module in workspace

```sh
go work init ./emojiis ./emoserv
```

## Publishing module

### Undestanding publishing

- Decentralized (Git)
- Module path: github.com/username/module

### Remote repository

- Repositoty: module root
- README.md
- LICENSE

### Local repository

- git
- Ssh key or gh cli

```sh
ssh-keygen -t ed25519 -C "name@mail.com"
cat ~/.ssh/id_ed25519.pub
```

### Importing remote modules

```sh
go mod tidy
```

### Documenting module

- Repository description
- Reposiroty topics
- README.md

### Discovering modules

[Pkg](pkg.go.dev)

## Module versioning

### Semantic versioning

- Immutable snapshots
- vMAJOR.MINOR.PATCH
- v0 is unstable
- suffix after patch number

### First unstable

```sh
go get module@0.1.0
```

### First stable

```sh

```

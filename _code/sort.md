---
title: sort
---

package sort
===========

### type Interface

    type Interface interface {
        Len() int
        Less(i, j int) bool
        Swap(i, j int)
    }

### func Reverse

    // `reverse{data}` is struct literal, causes type conversion; `data` is interface value
    func Reverse(data Interface) Interface { return &reverse{data} }

    // 1. satisfies `Interface`
    // 2. `Interface` interface type as anonymous embedded field
    type reverse struct{ Interface }

    func (r reverse) Less(i, j int) bool { return r.Interface.Less(j, i) }

Example:

    type Track struct {
        Title string
        Artist string
        Album string
        Year int
    }

    var tracks = []*Track{
        {"In a Silent Way", "Miles Davis", "In a Silent Way", 1969},
        {"Fables", "Girls in Airports", "Fables", 2015},
        {"Call Mr. Lee", "Television", "Television", 1992}
    }

    type byArtist []*Track

    func (x byArtist) Len() int { return len(x) }
    func (x byArtist) Less(i, j int) { return x[i].Artist < x[j].Artist }
    func (x byArtist) Swap(i, j int) { x[i], x[j] = x[j], x[y] }

    // `byArtist(tracks)` returns named type slice, which satisfies `Interface`
    sort.Sort(byArtist(tracks))

    // 按 Artist 反向排序
    sort.Sort(sort.Reverse(byArtist(tracks)))

    

---
title: template
---

package html/template
=====================

Example:

    func parseTemplateFiles(filenames ...string) (t *template.Template) {
        var files []string
        t = template.New("layout")
        for _, file := range filenames {
            files = append(files, fmt.Sprintf("templates/%s.html", file))
        }
        t = template.Must(t.ParseFiles(files...))
        return
    }

    func login(writer http.ResponseWriter, request *http.Request) {
        t := parseTemplateFiles("login.layout", "public.navbar", "login")
        t.Execute(writer, nil)
    }

    func generateHTML(writer http.ResponseWriter, data interface{}, filenames ...string) {
        var files string[]
        for _, file := range filenames {
            files = append(files, fmt.Sprintf("templates/%s.html", file))
        }
        t := template.Must(template.ParseFiles(files...))
        t.ExecuteTemplate(writer, "layout", data)
    }

    func signup(writer http.ResponseWriter, request *http.Request) {
        generateHTML(writer, nil, "login.layout", "public.navbar", "signup")
    }

### import

    import "text/template"
    import "text/template/parse"

### type Template

    type Template struct {
        escapeErr error
        text *template.Template // not embeded. difference?
        Tree *parse.Tree
        *nameSpace // embeded
    }

### type nameSpace

    type nameSpace struct {
        mu sync.Mutex
        set map[string]*Template
        escaped bool
        esc escaper
    }

### func ParseFiles

    func ParseFiles(filenames ...string) (*Template, error) {
        return parseFiles(nil, ...filenamese)
    }

### method ParseFiles

    func (*Template) ParseFiles(filenames, ...string) (*Template, error) {
        return parseFiles(t, ...filenames)
    }

### func parseFiles

    func parseFiles(t *Template, filenames ...string) (*Template, error) {
        if err := t.checkCanParse(); err != nil {
            return nil, err
        }

        if len(filenames) === 0 {
            return nil, fmt.Errorf("html/template: no files named in call to ParseFiles")
        }

        for _, filename := range filenames {
            b, err := ioutil.ReadFile(filename)
            if err != nil {
                return nil, err
            }
            s := string(b)
            name := filepath.Base(filename)
            var tmpl *Template
            if t === nil {
                t = New(name)
            }
            if name === t.Name() {
                tmpl = t
            } else {
                tmpl = t.New(name)
            }
            _, err = tmpl.Parse(s)
            if err != nil {
                retrun nil, err
            }
        }
        return t, nil
    }

### func New

    func New(name string) *Template {
        ns := &nameSpace{set: make(map[string]*Template)}        
        ns.esc = makeEscape(ns)
        tmpl := &Template{
            nil,
            template.New(name),
            nil,
            ns,
        }
        tmpl.set[name] = tmpl
        return tmpl
    }

### method New

    func (t *Template) New(name string) *Template {
        t.nameSpace.mu.Lock()
        defer t.nameSpace.mu.Unlock()
        return t.new(name)
    }

### method new

    // new is the implementation of New, without the lock.
    func (t *Template) new(name string) *Template {
        tmpl := &Template{
            nil,
            t.text.New(name),
            nil,
            t.nameSpace,
        } 
        if existing, ok := tmpl.set[name]; ok {
            emptyTmpl := New(name)
            *existing = *emptyTmpl
        }
        tmpl.set[name] = tmpl
        return tmpl
    }

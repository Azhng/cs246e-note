# Probelm 21 - The copier is broken 

How do copies & moves interact with inheritance 

Copy ctor: 
- `Textbook::Textbook(const Textbook &other) : Book{other}, topic{other.topic} {}`

Move ctor: 
- `Textbook::Textbook(Textbook &&other) : Book{std::move(other)}, topic{std::move{other.topic}} {}`

Copy/move assignment:
``` C++
Textbook& Textbook::operator=(Text other) {
    Book::operator=(std::move(other));
    topic = std::move(other.topic);
    return *this;
}
```

But consider:
``` C++
Book *b1 = new Text{"Basic", "", "", "Some dude"}, *b2 = new Text{"C++", "", "", "Stroustrap"};
(*b1) = (*b2);
// What happens 
```
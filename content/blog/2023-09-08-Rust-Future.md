+++
title = "Why doesn't my future became !Send?"
[taxonomies]
  tags = ["rust","async"]
[extra]
  toc = true
+++

## Background

Recently, I face a problem when i use `auxm` http framework, which is

```rust
the trait `Handler<_, _, _>` is not implemented for fn item
```

after i using [auxm-debugging-marcos](https://docs.rs/axum/latest/axum/handler/index.html#debugging-handler-type-errors). I fanlly found the probelm is `Future is not Send`.Bacause is auxm the `Handler` tarit, we can see the tarit has a **asoccsiation type** Futrue need to be **Send**

```rust
pub trait Handler<T, S, B = Body>: Clone + Send + Sized + 'static {
    type Future: Future<Output = Response> + Send + 'static;

    // Required method
    fn call(self, req: Request<B>, state: S) -> Self::Future;

    // Provided methods
    fn layer<L, NewReqBody>(
        self,
        layer: L
    ) -> Layered<L, Self, T, S, B, NewReqBody>
       where L: Layer<HandlerService<Self, T, S, B>> + Clone,
             L::Service: Service<Request<NewReqBody>> { ... }
    fn with_state(self, state: S) -> HandlerService<Self, T, S, B> { ... }
}
```

I wandering why my async function return a `!Send Futrue`.So I'm starting to dig debug my code. After a while, I found the bug in my code:

Bacause I tried to make `parse_statement` to be a async function, but in this function the variable `Parser`  has a field `RecursionCounter` which is has a field `Rc<AtomicUsize>` !!. As all we known `Rc` is not Send. So that's make `parse_statement` return a Futrue is not Send.

```rust
pub async fn parse_statement(&mut self, sql: &str) -> Result<Statement> {
        ...
        let mut parser = Parser::new(dialect).with_tokens_with_locations(tokens);
        ...
}

pub struct Parser<'a> {
    ...
    recursion_counter: RecursionCounter,
}

pub(crate) struct RecursionCounter {
    remaining_depth: Rc<AtomicUsize>,
}
```

The 'bug' is really easy to fix. We just need to split the async code and non-async code so that we can avoid returning a Future that is not Send. After I solved the 'bug', I still didn't know why this bug occurred so I decided to write this blog to figure out the relationship between Send, !Send, and Future.

That's base from those questions:

- What's the Future?
- What's the Send?
- Why Future not Send?

## What's the Future?

## What's the Send?

[understanding-the-send-trait](https://stackoverflow.com/questions/59428096/understanding-the-send-trait)

## Why Future not Send?

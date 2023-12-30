- try-cath문 사이에서 사용해서는 안된다.
- 만약 try 성공 시 사용하고 싶다면? -> 따로 변수를 사용

```ts
let shouldRedirected = false;

try {
	// ...
	shouldRedirected = true;
} catch(err) {
	// ...
}

if (shouldRedirected) {
	redirect('...');
}
```
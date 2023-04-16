# You Don’t Need a Reactive Template Library

In web development, the norm is to rely on reactive template libraries, as if implementing this functionality was beyond the realm of what is practical for you, the humble web developper, to do on your own.

These libraries, the grand pantheon of AngularJS, Alpine, Vue, and React, preside over us mere mortals like enigmatic gods, their inner workings veiled in a fog of arcane mystery.

Well your salvation has arrived in the form of 10 lines code that will render your bloated, ostentatious libraries obsolete.

First the template we will be rendering:

```html
<jodie-view>
	<div data-for="item of data" id="item-${item.title}">
		<b>${item.title}</b><br>
		<div data-for="element of item.elements">
			${element}<br>
			<a href="${element}">link</a><br>
		</div>
	</div>
</jodie-view>
```

and some demo data:

```js
data = [
	{title:"one",elements:["foo","bar"]},
	{title:"two",elements:["lorem","ipsum"]},
	{title:"three",elements:["hello","world"]},
	{title:"four",elements:["foo","bar"]},
	{title:"five",elements:["lorem","ipsum"]},
];
```
Time to show the world a new way to live, free from the clutches of over-engineered solutions. Behold as this tiny render function dares to paint the initial page without any diffing.

```js
let JVPlugins = [
	["[data-for]",(e)=>`\${[...(function *(){for(let ${e.dataset.for})yield\`${(delete e.dataset.for,e.outerHTML)}\`})()].join("")}`],
];

class JodieView extends HTMLElement{
	connectedCallback(){
		for(const [selector,transform] of JVPlugins)
			for(let e;e=this.querySelector(selector);)
				e.outerHTML = transform(e);
		this.template = Function(`return\`${this.innerHTML}\``);
	}
	render(){
		this.innerHTML = this.template();
	}
}

customElements.define("jodie-view",JodieView);
```
## Isn't rerendering going to be broken and slow without diffing?

The code's masterstroke is its ability to patch in fancy diffing, only after the page has painted. Many libraries do this already, like the excellent https://diffhtml.org/
```js
import("https://cdn.jsdelivr.net/npm/diffhtml@1.0.0-beta.29/+esm").then(({innerHTML})=>{
	JodieView.prototype.render = function(){
		innerHTML(this,this.template());
	};
});
```
or https://github.com/patrick-steele-idem/morphdom

```js
import("https://cdn.jsdelivr.net/npm/morphdom@2.7.0/+esm").then(({default:morph})=>{
	JodieView.prototype.render = function(){
		let div = document.createElement("div");
		div.innerHTML = this.template();
		morph(this,div,{childrenOnly:true});
	};
});
```
Server-side rendering and fancy hydration have long been hailed as the holy grail for optimizing performance and user experience. However, when armed with a templating engine so tiny, these techniques suddenly lose their appeal. I hope I've shown don't need to sacrifice the convenience of static websites on the altar of first-paint performance, have your cake and eat it too, you're welcome.


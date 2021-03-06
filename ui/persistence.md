# Persistence

Persistence refers to the app storing the state of an object as an instance of it is removed and recreated. In practice this means sharing

## Persistence via routing

URL is a good place to store some data, as it can be bookmarked, shared between users etc. Obviously this has many limitations, but in many cases this is the best place to store information such as current page, current tab or an ID of an object whose information you're showing on the page.

## Persistence in components

[Components](./app/components.md) can easily store their state in local storage. This is implemented with a mixin, which tracks a computed parameter `persist` and stores it whenever it changes. When the component is recreated, the mixin loads values from local storage and updates the newly created instance to use those values (as defined by the `persist` setter).

In order for persistence to work, we need to have a key to store the persistent values with. To make sure each component has one, you **must** define either the component's `name` property (which you should always do anyway) or a _computed_ property `persistKey` for your component in order for peristence to work.

Any two Vue objects with the same `persistKey` (which defaults to the object's `name`) will **share the same persistent values**. This means that by default, persistence is shared across all instances of the same component. When authoring your code, be aware of this so that persistence works as expected for your users.

Persistence in components is implemented via a global mixin. Here is a usage example:

```js
// MyPersistentComponent.vue
import persist from '@mixins/persist'

export default {
	name: 'foo-component',

	mixins: [persist],

	data: function () {
		return {
			someValue: 'Foo'
		};
	},

	computed: {
		persist: {

			// The mixin adds a watcer for this
			// Whenever this value changes, it is stored in local storage
			get: function () {
				return {
					someValue: 'Foo'
				};
			},

			// Called when data is loaded
			// This is where you define how you treat the stored data
			// This could be more complicated, for example you could decide to load the values only with current route
			set: function (stored) {
				if (stored.someValue) {
					this.someValue = stored.someValue;
				}
			}

		}
	}
}
```

You can view the source for the global mixin on [GitHub](https://github.com/Eiskis/bellevue/tree/master/src/mixins/persist.js) to see what behavior it attaches to the components.

Note that persistent values **are not** synced real-time, i.e. persistent data is not updated in other instances of a component as it changes - only stored to local storage and used the next time a component instance with the same `persistKey` is created. If you want to share state, use [services](../app/services.md) or [Vuex](../app/vuex.md) instead.

- [Read more about computed setters](https://vuejs.org/guide/computed.html#Computed-Setter)
- [Read more about mixins](https://vuejs.org/v2/guide/mixins.html)

## Persistence in services

[Services](../app/services.md) can also store their state in local storage in the same way. However, as services do not have access to the root Vue instance, they do not have access to plugins (e.g. `this.$route`).

If you're in a situation where you want to do this, you can persist values in the `<App>` component, which can then call logic you have in services.

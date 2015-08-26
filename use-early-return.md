## Bad code

```js
{
  filteredGroups: Ember.computed('groups.[]', 'filterText', function () {
    let groups = this.get('groups');
    let filterText = (this.get('filterText') || '').toLowerCase();

    if (filterText) {
      groups = groups.filter(group => group.name.toLowerCase().includes(filterText));
    }

    return groups;
  })
}
```

Problem: I need to read through all the code to see that when filter text is empty, this no operation function.

## Better code

Lets start with simple refactoring, return as early as possible. Assuming that `filterText` is not empty, you can see that the code first assigns `group`, then immediately returns it. Assignment is redundant here, lets return early instead of it.

```js
{
  filteredGroups: Ember.computed('groups.[]', 'filterText', function () {
    let groups = this.get('groups');
    let filterText = (this.get('filterText') || '').toLowerCase();

    if (filterText) {
      return groups.filter(group => group.name.toLowerCase().includes(filterText));
    }

    return groups;
  })
}
```

Step 2

```js
{
  filteredGroups: Ember.computed('groups.[]', 'filterText', function () {
    let groups = this.get('groups');
    let filterText = (this.get('filterText') || '').toLowerCase();

    if (!filterText) {
      return groups;
    }

    return groups.filter(group => group.name.toLowerCase().includes(filterText));
  })
}
```

Note: we don't need to convert empty string to lower case to check that it is empty, so we can simplify code before early return even more.

```js
{
  filteredGroups: Ember.computed('groups.[]', 'filterText', function () {
    let groups = this.get('groups');
    let filterText = this.get('filterText');

    if (!filterText) {
      return groups;
    }
    
    filterText = filterText.toLowerCase();

    return groups.filter(group => group.name.toLowerCase().includes(filterText));
  })
}
```

## Next steps

* remove variables, use `get` inline
* extract logic from filter functional exptession into `match` function

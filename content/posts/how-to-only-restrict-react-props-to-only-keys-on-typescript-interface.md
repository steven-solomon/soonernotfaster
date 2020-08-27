---
title: "How to Only Restrict React Props to Only Keys on Typescript Interface"
date: 2020-08-23T14:07:40-04:00
draft: true
summary: 'wibble that wobble'
---

```typescript
interface Fish {
  species: string;
  color: Color;
}

const fishPart: keyOf Fish = "wibble" // generates error
const fishPart: keyOf Fish = "color" // works just fine
```

Building input forms is so often a rote task, reading from selectors to display data, and dispatching actions, leaving us wondering "how might we eliminate this tedious work, while restricting our types to the fields on our interface?" 

Thankfully TypeScript takes the issue of data integrity from the runtime world of JavaScript, and brings into our editor—using a plugin—to give us feedback while we are writing our program.

Consider the example of a form that allows users to enter the details of their new pet fish. In this example we have data for our fish stored in a redux store and across several views we want to dispatch updateFish actions to our store. This quickly becomes a tedious process as we end up with thirty fields for our Fish. There has got to be a better way!

In true lean fashion, we identify that we are continually building an input component that both reads data from a redux store, and dispatches change actions. This motion waste can be eliminated if we had a single representation of this construct in our system.

Below is the code that exists throughout our system:

```typescript
const Form = () => {
  const name = useSelector<ApplicationState, string | undefined>((state) => state.name)
  const finColor = useSelector<ApplicationState, string | undefined>((state) => state.finColor)
  const favoriteHidingSpot = useSelector<ApplicationState, string | undefined>((state) => state.favoriteHidingSpot)

  const dispatch = useDispatch<(action: FishAction) => void>()

  return (
    <form>
      <input type='text' onChange={(event) => {
        dispatch({type: ActionType.setName, name: event.target.value});
      }} value={name} />
      <input type='text' onChange={(event) => {
        dispatch({type: ActionType.setFinColor, finColor: event.target.value});
      }} value={finColor} />
      <input type='text' onChange={(event) => {
        dispatch({type: ActionType.setFavoriteHidingSpot, favoriteHidingSpot: event.target.value});
      }} value={favoriteHidingSpot} />
      <div>
        <h1>Fish profile</h1>
        <p>Name: {name}</p>
        <p>Fin color: {finColor}</p>
        <p>Favorite Hiding Spot: {favoriteHidingSpot}</p>
      </div>
    </form>
  )
}
```

Removing the need to reconstruct this useful widget for each user story would improve our throughput as a team but there is a problem. Constructing a component that dynamically sets and reads from a field can generate a new type of bug in our system, the field name can’t be checked until runtime. 

However, there is a solution! The `keyof` operator. Adding this to our component interface will allow us to be sure that we can prevent silly issues. Checkout out the solution below:

```
Code example with keyof in interface
``` 
---
title: "Making Invalid State Impossible: in TypeScript and React"
featured_image: "images/making_invalid_state_impossible_in_typescript_and_react.jpeg"
images: ["images/making_invalid_state_impossible_in_typescript_and_react.jpeg"]
date: 2018-10-29T21:14:59-04:00
---

Since most applications can be represented by state machines, it is beneficial to make valid states in your program as clear as possible and invalid states impossible. This will enhance the readability, and maintainability of your code to everyone who sees it, including future you.

In the rest of this article we will discuss one strategy for making invalid states impossible with an example in tsx and React.

# Replacing Booleans With Types

Below you will find a tiny React component that displays fish from a fictional API. You will notice that the component handles both failure and success: on success, it displays the fish; on failure it displays an error message. There is also a state representing when the component is first waiting for a response.

These states are represented by a pair of booleans, one for success and one for failure.

```tsx
import * as React from 'react'
import axios from 'axios'

interface Fish {
  name: String
}

interface FishTankState {
  fish?: [Fish]
  isError: boolean // state as boolean
  isSuccess: boolean
}

export class FishTank extends React.Component<{}, FishTankState> {
  state: FishTankState = {
    fish: null,
    isError: false,
    isSuccess: false,
  }

  componentDidMount() {
    axios
      .get<[Fish]>('fish')
      // setting state
      .then(f => this.setState({isSuccess: true, fish: f.data}))
      .catch(() => this.setState({isError: true}))
  }
  render() {
    // check state
    if (this.state.isSuccess) {
      return this.state.fish.map((f, i) => <div key={i}>{f.name}</div>)
    } else if (this.state.isError) {
      return <>Could not load fish</>
    } else {
      return <></>
    }
  }
}
```

Though it is straight forward, this code has a weakness, it is a possible to create an invalid state. The case when `isError` and `isSuccess` are both true. While there is no way at the moment to push the code into this combination, it may be a stumbling block for an engineers in the future.

*It would be best if we instilled the valid states into the source code using types, so that invalid states are impossible.*

First, we will create a new enum `requestStatus` to represent the states of the component. We will slowly move the existing code from using the pair of booleans to the enum. The structures will live in parallel for a time, allowing the render method to lean more and more on the enum, until the refactoring is complete.

We begin by adding the initial “Not Loaded” state, and adding extract checks on the booleans so that the tests pass.

```tsx
import * as React from 'react'
import axios from 'axios'

interface Fish {
  name: String
}

// Added enum
enum RequestStatus {
  NotLoaded,
}

interface FishTankState {
  fish?: [Fish]
  isError: boolean
  isSuccess: boolean
  requestStatus: RequestStatus
}

export class FishTank extends React.Component<{}, FishTankState> {
  state: FishTankState = {
    fish: null,
    isError: false,
    isSuccess: false,
    requestStatus: RequestStatus.NotLoaded, // initial state
  }

  componentDidMount() {
    axios
      .get<[Fish]>('fish')
      .then(f => this.setState({ isSuccess: true, fish: f.data }))
      .catch(() => this.setState({ isError: true }))
  }
  render() {
    // begin to use NotLoaded state
    if (
      this.state.requestStatus === RequestStatus.NotLoaded &&
      this.state.isError === false &&
      this.state.isSuccess === false
    )
      return <></>

    if (this.state.isSuccess) {
      return this.state.fish.map((f, i) => <div key={i}>{f.name}</div>)
    } else if (this.state.isError) {
      return <>Could not load fish</>
    } else {
      return <></>
    }
  }
}
```

Now that the initial state of the component is being used to render the “Not Loaded” state, we can attach another state to the enum, “Success”.

Additionally, inside of the `then` callback for the API request, we will transition the `requestStatus` to the “Success” state.

```tsx
import * as React from 'react'
import axios from 'axios'

interface Fish {
  name: String
}

enum RequestStatus {
  NotLoaded,
  Success, // Added
}

interface FishTankState {
  fish?: [Fish]
  isError: boolean
  isSuccess: boolean
  requestStatus: RequestStatus
}

export class FishTank extends React.Component<{}, FishTankState> {
  state: FishTankState = {
    fish: null,
    isError: false,
    isSuccess: false,
    requestStatus: RequestStatus.NotLoaded,
  }

  componentDidMount() {
    axios
      .get<[Fish]>('fish')
      .then(f =>
        this.setState({
          isSuccess: true,
          requestStatus: RequestStatus.Success, // Added
          fish: f.data,
        })
      )
      .catch(() => this.setState({ isError: true }))
  }
  render() {
    if (
      this.state.requestStatus === RequestStatus.NotLoaded &&
      this.state.isError === false &&
      this.state.isSuccess === false
    )
      return <></>
    // Check on new state
    else if (this.state.requestStatus === RequestStatus.Success)
      return this.state.fish.map((f, i) => <div key={i}>{f.name}</div>)

    if (this.state.isSuccess) {
      return this.state.fish.map((f, i) => <div key={i}>{f.name}</div>)
    } else if (this.state.isError) {
      return <>Could not load fish</>
    } else {
      return <></>
    }
  }
}
```

With Success state using the new requestStatus variable, we now transition the last remaining valid state to our enum, Error.

```tsx
import * as React from 'react'
import axios from 'axios'

interface Fish {
  name: String
}

enum RequestStatus {
  NotLoaded,
  Success,
  Error, // Added
}

interface FishTankState {
  fish?: [Fish]
  isError: boolean
  isSuccess: boolean
  requestStatus: RequestStatus
}

export class FishTank extends React.Component<{}, FishTankState> {
  state: FishTankState = {
    fish: null,
    isError: false,
    isSuccess: false,
    requestStatus: RequestStatus.NotLoaded,
  }

  componentDidMount() {
    axios
      .get<[Fish]>('fish')
      .then(f =>
        this.setState({
          isSuccess: true,
          requestStatus: RequestStatus.Success,
          fish: f.data,
        })
      )
      .catch(() =>
        // Set Error state
        this.setState({ isError: true, requestStatus: RequestStatus.Error })
      )
  }

  render() {
    if (
      this.state.requestStatus === RequestStatus.NotLoaded &&
      this.state.isError === false &&
      this.state.isSuccess === false
    )
      return <></>
    else if (this.state.requestStatus === RequestStatus.Success)
      return this.state.fish.map((f, i) => <div key={i}>{f.name}</div>)
    else return <>Could not load fish</>

    if (this.state.isSuccess) {
      return this.state.fish.map((f, i) => <div key={i}>{f.name}</div>)
    } else if (this.state.isError) {
      return <>Could not load fish</>
    } else {
      return <></>
    }
  }
}
```

With this new structure, the tests still pass, and we can delete the old booleans and the code that relies on them. Leaving the code in this final state.

```tsx
import * as React from 'react'
import axios from 'axios'

interface Fish {
  name: String
}

enum RequestStatus {
  NotLoaded,
  Success,
  Error,
}

interface FishTankState {
  fish?: [Fish]
  requestStatus: RequestStatus
}

export class FishTank extends React.Component<{}, FishTankState> {
  state: FishTankState = {
    fish: null,
    requestStatus: RequestStatus.NotLoaded,
  }

  componentDidMount() {
    axios
      .get<[Fish]>('fish')
      .then(f =>
        this.setState({
          requestStatus: RequestStatus.Success,
          fish: f.data,
        })
      )
      .catch(() => this.setState({ requestStatus: RequestStatus.Error }))
  }

  render() {
    if (this.state.requestStatus === RequestStatus.NotLoaded) 
      return <></>
    else if (this.state.requestStatus === RequestStatus.Success)
      return this.state.fish.map((f, i) => <div key={i}>{f.name}</div>)
    else return <>Could not load fish</>
   }
  }
}
```

One Last stylistic addition, would be to make the states clear in the `render` method by replacing the if-else-chain with a switch-statement.

```tsx
//...
render() {
  switch (this.state.requestStatus) {
    case RequestStatus.NotLoaded:
      return <></>
    case RequestStatus.Error:
      return <>Could not load fish</>
    case RequestStatus.Success:
      return this.state.fish.map((f, i) => <div key={i}>{f.name}</div>)
  }
}
//...
```

# Conclusion

Invalid states can be a stumbling block for those modifying your code. Making the transitions to invalid states impossible will go a long in enhancing the readability of your code.

Remember to move states to one place in the program, find ways to communicate which states are mutually exclusive, and eliminate unnecessary states from your program.

{{< thank-you >}}

*Photo by Malcolm Lightbody on Unsplash*
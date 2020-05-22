---
layout: post
title: "Exploring MVU Par 2: Composition"
categories: Posts
author: Austin Webre
---

So a couple days have come and gone and I'm still riding high on my experience with the MVU pattern. I'm struggling some with the syntax of F#, but I'm getting it bit by bit and even learning to love some aspects of it. That being said, the sample app I went through last time did nothing but peak my interest. It felt really smooth and easy to reason about, but "Counters" always are, right? So the next logical question is, how do we make this work for a more complicated situation?

I did some digging, but all the examples I could find simply stuffed more code into the view. Which is fine for smaller tasks, but thats just not sustainable for an app with multiple pages and navigation. So I started googling but ended up going in all the wrong directions. Eventually, I came across something in the docs for Elmish (an F# MVU-like framework) that really opened things up. In the example, there's a `Counter` module that is used by a `CounterList` module which reuses the `Counter`'s model, update, and view logic. I could post the example and we could go through it, but I thought this might be a good chance to make some progress on `commutr_v2`.

I'm going to start by trying to recreate the first page of the `commutr` app, which displays all the vehicles that you can track. Now in a future version of this app, we're going to skip this step if the user has marked a "Primary" vehicle. So really, this work is mostly going to be useless if it ever gets off the ground. But its a good simple case and a good first place to start. My first instinct is to follow the counter example I explained above. So we'll start with the smallest part first.

# VehicleItem.fs

```fsharp
namespace commutr_v2

open Fabulous
open Fabulous.XamarinForms
open Xamarin.Forms

module VehicleItem =
    type Model =
      { Id : int
        Make : string
        Model : string
        Year : int
        IsPrimary : bool }

    type Msg =
        | TogglePrimary of bool

    let init (initModel : Model) = initModel

    let update msg model =
        match msg with
        | TogglePrimary isPrimary -> { model with IsPrimary = isPrimary }

    let view (model: Model) dispatch =
        View.StackLayout(padding = Thickness 20.0, verticalOptions = LayoutOptions.Center, orientation = StackOrientation.Horizontal,
            children = [
                View.Label(text = model.Make, horizontalOptions = LayoutOptions.Center, width=200.0, horizontalTextAlignment=TextAlignment.Center)
                View.Label(text = model.Model, horizontalOptions = LayoutOptions.Center, width=200.0, horizontalTextAlignment=TextAlignment.Center)
            ])
```

This is pretty straight forward, so I won't go in too much detail. The plan here is to output a view that can be used as a child of a `CollectionView`. So we have our model which represents a vehicle, the update which can set whether the vehicle in question is primary, and the view which puts it all together. We may come back to this later, as I'm not entirely sure about the rules of what we can and can't return from the view, but so far it appears that we can return any UI Element whatsoever, so we'll keep operating under that assumption.

Now that we have the smallest piece set up, lets move one step higher.

# VehicleListing.fs

```fsharp
module VehicleListing =
    type Model = {
        Vehicles : VehicleItem.Model list
        SelectedVehicle : Option<VehicleItem.Model>
        }

    type Msg =
        | Insert
        | Remove
        | Modified of int * VehicleItem.Msg
        | SelectVehicle of int

    let initModel : Model = { Vehicles = [
                            {Id = 1; Make = "Honda"; Model = "Accord"; Year = 2005; IsPrimary = false}
                            {Id = 2; Make = "Honda"; Model = "Insight"; Year = 2019; IsPrimary = true}
                            ];
                            SelectedVehicle = None }

    let init() = initModel

    let update msg model =
        match msg with
        | Insert -> model //TODO: Make this do something
        | Remove -> model //TODO: Make this do something
        | Modified (pos, itemMessage) -> { model with Vehicles = model.Vehicles
                                                        |> List.mapi (fun i itemModel ->
                                                        if i = pos then
                                                            VehicleItem.update itemMessage itemModel
                                                        else
                                                            itemModel)}
        | SelectVehicle pos -> {model with SelectedVehicle = Some(model.Vehicles.Item(pos))}

    let view (model : Model) dispatch =
        let items = model.Vehicles |> List.mapi(fun pos itemModel ->
                             VehicleItem.view
                                itemModel
                                (fun msg -> dispatch (Modified (pos, msg))))
        View.CollectionView(items)
```

Now this is clearly a work in progress, but I wanted to map out where I'm heading while simultaneously getting the most simple case working first. There are (once again) three novel things that are happening here. The first is that part of the Model for this module is a list of models from the `VehicleItem` module. Model, module, model, module... bleh.

Anyway, the second is in the update. You'll notice the arguments for the `Modified` case are `pos` (an integer representing the index of the item in the array) and `itemMessage` which is of type `VehicleItem.Msg`. Those arguments are then used to update the model that matches the correct position using `VehicleItem.update`. So all we're doing is routing the message back to the appropriate update function. And since our `update` functions are pure, we don't need a specific instance or anything, just pass the model and the message.

The last novel thing is in the view (you guessed it). We get a list of items by mapping the List of `VehicleItem.Model` in our model to a list of Stack Layout Views via the `VehicleItem.view` function. There's something really interesting that we're doing here that makes this work. We're hijacking the `dispatch` function so that the `Modified` message gets sent instead. Its then up to our `Modified` message to update the appropriate item.

Hopefully that makes sense.

Now, you may notice that I defined a `SelectVehicle` message that will handle selection. I did this mostly just to work out how that might hypothetically look. It isn't used here because I realized that `CollectionView` requires you to wrap each item in a `SwipeView`, so we may have to move that logic down some how. Or maybe not. I haven't gotten that far yet.

Back to making the simplest case work.

# Wiring It All Up

So now that we have the Vehicle List worked out, lets tie it all together.

```fsharp
namespace commutr_v2

open System.Diagnostics
open Fabulous
open Fabulous.XamarinForms
open Fabulous.XamarinForms.LiveUpdate
open Xamarin.Forms

module App =
    type Model =
      { Vehicles : VehicleListing.Model }

    type Msg =
        | VehicleListUpdated of VehicleListing.Msg
        | RefreshVehicles

    let initModel = { Vehicles = VehicleListing.init() }

    let init () = initModel, Cmd.none

    let update msg model =
        match msg with
        | VehicleListUpdated listMsg -> {model with Vehicles = VehicleListing.update listMsg model.Vehicles}, Cmd.none
        | RefreshVehicles -> {model with Vehicles = VehicleListing.init()}, Cmd.none

    let view (model: Model) (dispatch : Dispatch<Msg>) =
        let vehicleList = VehicleListing.view
                            model.Vehicles
                            (fun msg -> dispatch (VehicleListUpdated msg))
        View.ContentPage(
          content = View.StackLayout(padding = Thickness 20.0, verticalOptions = LayoutOptions.Center,
            children = [
                vehicleList
            ]))

    // Note, this declaration is needed if you enable LiveUpdate
    let program = XamarinFormsProgram.mkProgram init update view

type App () as app =
    inherit Application ()

    let runner =
        App.program
#if DEBUG
        |> Program.withConsoleTrace
#endif
        |> XamarinFormsProgram.run app
```

So this is not all that unsimilar to the `VehicleListing`. We make the child model part of our model and pipe the child's update and view logic into our own. The result isn't the prettiest, but it displays, so... a job well done?

![Wired Up UI](/assets/posts/mvu-composition/wired-up-ui.png)

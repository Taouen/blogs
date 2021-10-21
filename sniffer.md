https://m2m.pqsec.org/

I’ve been working on a companion app for a trading card game called Magic: The Gathering (or MTG for short). In the game, two or more players spend resources called mana to cast spells trying to reduce their opponents’ life total from 20 to 0. The app displays possibilities for cards an opponent could play, based on an amount of mana entered by the user. To understand the problem, we’ll start with some context.

There are 5 colors of mana, each represented in shorthand by a single letter: white (W), blue (U), black (B), red (R), and green (G). There is also a sixth type of mana called colorless (or C), though it doesn’t come up as commonly. Cards that can produce mana are called mana sources, and can typically produce one mana of one or more colors (depending on the source) per turn cycle (from the start of a player’s turn until the start of their next turn).

Most spells can only be played during the “Main Phase” of your turn, but some can be played at any time during any player’s turn. These spells are referred to as being “instant speed”, and they are the spells we are interested in for this app.

Each spell has a mana cost, which indicates how much and what colors of mana are required to cast it. Generic mana costs are the most common. They are simply represented by a number, and can be paid with any of the 5 colors of mana, or colorless. The colored mana is denoted by mana symbols on cards, and by the letters listed above wrapped in curly braces for text. For example, a cost of 1{R} requires 2 total mana, one of which must be red, and a cost of 3{U}{B} requires 5 total mana, one of which must be blue, and one which must be black.

The app allows the user to enter mana of any one of the 5 colors, or of any color (the gold symbol shown below), or colorless. The mana filter is created by looping over a state object, returning a `ManaButton` component for each color.

```javascript
mana: {
  W: 0,
  U: 0,
  B: 0,
  R: 0,
  G: 0,
  M: 0, // M represents sources that produce any color of mana
  C: 0
}
```

![manafilter1.png](./assets/manafilter1.png 'First version of mana filter')

<br>

This covers most situations you’d come across. However, what if your opponent has a land that can produce both blue and black mana? This was the challenge I was faced with: add a way for the user to create a new mana source button with colors that they select, and have the app display cards appropriately.

This problem can be broken into three parts:

1. Select colors and create a new mana source button.
2. Allow custom multicolor mana sources to be used for any of their defined colors.
3. Allow custom multicolor mana sources to be used only as many times as the user indicates are available (ie. a source that creates blue or black should only count for 1 mana and not 2).

### Select colors and create a new mana source button

The easiest part of this problem was the UI. I knew I wanted to have some kind of circular menu that presented the 5 colors where the user could select 2-4 colors, and then confirm or cancel.

I looked at a few different open source libraries for creating circular menus, and I landed on [react-planet](https://github.com/innFactory/react-planet) as it was more flexible and visually pretty much what I wanted.

I created an state object within my multicolorMenu component called colors, and listed the 5 main colors that I'd need to make buttons for.

```javascript
const [colors, setColors] = useState({
  W: { value: 'W', symbol: W, selected: false, sortOrder: 1 },
  U: { value: 'U', symbol: U, selected: false, sortOrder: 2 },
  B: { value: 'B', symbol: B, selected: false, sortOrder: 3 },
  R: { value: 'R', symbol: R, selected: false, sortOrder: 4 },
  G: { value: 'G', symbol: G, selected: false, sortOrder: 5 },
});
```

<br>

Using Object.entries(), I mapped over the colors object and created buttons for each one.

![multicolormenu.png](./assets/multicolormenu.png 'Multicolor selection menu')

<br>

A few things happen when one of the buttons is clicked. First, the selected value for that color is changed to true so that I can conditionally display a checkmark over the button indicating that it's selected. Second, the color object is added to a `colorsForNewManaSource` state array, and then the array is sorted by each object's `sortOrder` value. This is important later on.

Once the user has selected between 2 and 4 colors, the menu's close button switches to a confirm button with a checkmark. The menu won't allow a user to confirm with a single color or all 5 colors selected because they are already available in the standard mana filter. When the user confirms the selections, the `value` of each object in the `colorsForNewManaSource` array are joined together into a single string. Remember how the color objects were sorted by their `sortOrder` properties? This is important because it creates consistency in the naming convention, so when we pass the name to the ManaButton component, it can find the symbol svg with the same name exported from the symbols.js file. The name is then used to create a new source in the app's main mana state.

```js
mana: {
  W: 0,
  U: 0,
  B: 0,
  R: 0,
  G: 0,
  M: 0, // M represents sources that produce any color of mana
  C: 0,
  WU: 0
}
```

<br>

Here's the final result, with the app rendering after the change to the mana state:

#### Input video here

### Allow custom multicolor mana sources to be used for any of their defined colors

Now that the user can create a button for a new multicolor mana source, I needed to make it do something. To start, I needed the new mana source to be able to allow cards of any of its colors to be displayed, so I needed to figure out how to identify what the colors are in the first place.

For my first pass at this, I tried to use the name of the color (the string value we created when confirming the source) by just using the string `split()` method. In the interest of not repeating my code, I needed to use the same method that I did for the rest of the mana sources. However, this method fails when it reaches the any color mana (M), since it is represented by a single letter.

I decided to refactor the mana state. Instead of the value for each property being simply a number representing the available amount of that color, I changed it to an object so that I could add an array of colors that it could produce.

```js
mana: {
      W: { value: 0, colors: ['W'] },
      U: { value: 0, colors: ['U'] },
      B: { value: 0, colors: ['B'] },
      R: { value: 0, colors: ['R'] },
      G: { value: 0, colors: ['G'] },
      M: { value: 0, colors: ['W', 'U', 'B', 'R', 'G'] },
      C: { value: 0, colors: [] },
    }
```

<br>

Now when a new source is created, the colors value for that object is created by using the `split()` method. So the new source added to the mana state looks like this:

```js
WU: { value: 0, colors: ['W', 'U']},
```

### Notes

- The problem: how to allow a user to account for a mana source that can provide more than one color of mana.
  Breaks down into smaller challenges:

1. Allow the user to create a new mana source with selected colors.

   - Create a new manaButton component, and find the correct symbol for the selected colors

   - Tried creating a multicolorMana state object to store any multicolor sources. This became complicated, trying to convert the functions that handle mana changes needing to have multicolorMana added in as well.
   - Refactored mana state values from just the value (number) to an object containing the value and an array of colors that the source can produce. This made things simpler, allowing me to combine all the mana sources into the mana state, simplify functions dealing with mana objects (manaButton, mana change handlers, etc) and eliminate duplicate functions (ex. I had addManaSource and addMulticolorManaSource methods in App.js). Able to just pass the info to handlers, standardizing it with only the colors produced and length of the name (key) of the mana source making them different.
   - This also simplified the creation of the manaButton components. I was able to loop over a single array and create buttons for each source in the list, just adding the new ones as they were created.
   - Used react-planet to create the multicolor menu.
   - When user selects a color, it is pushed to a temporary array. When more than one item is selected, they are sorted in WUBRG order. In this way, no matter what order the colors are selected in, the order of the string output will be consistent, which is necessary for it to find the correct mana symbol svg.

2. Allow a multicolor source to be used for any of its defined colors.
   - Tried to use just the name of the source (‘WUB’, ‘RG’, ‘GU’, etc). Too complicated.
   - Add colors array to mana state to provide a way to check the colors of a source.
   - Prioritize sources that produce fewer colors (create array copy of mana state to preserve order, sorted by number of colors produced)
3. Allow a multicolor source to be used only as many times as are available as indicated by the user.
   - Loop over required mana, and compare each to each mana source.
   - If the source produces that color of mana, and has a value > 0, use that mana, then move on (either to the next source if the requirement for the color is unfulfilled or to the next required color if it is)

https://m2m.pqsec.org/

I’ve been working on a companion app for a trading card game called Magic: The Gathering (or MTG for short). In the game, two or more players spend resources called mana to cast spells trying to reduce their opponents’ life total from 20 to 0. The app displays possibilities for cards an opponent could play, based on an amount of mana entered by the user. To understand the problem, we’ll start with some context.

There are 5 colors of mana, each represented in shorthand by a single letter: white (W), blue (U), black (B), red (R), and green (G). There is also a sixth type of mana called colorless (or C), though it doesn’t come up very commonly. Cards that can produce mana are called mana sources, and can typically produce one mana of one or more colors per turn cycle (from the start of a player’s turn until the start of their next turn).

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

manafilter1.png

<br>

When the user adds any amount of mana, the app goes through the selected set of cards, comparing the cards required amounts of each color of mana with the available amounts entered by the user. If the required amount of each color of mana is available and the total cost of the card is less than or equal to the total available mana, then the card is added to the list of cards to be displayed.

This covers most situations you’d come across. However, what if your opponent has a land that can produce both blue and black mana? This was the challenge I was faced with: add a way for the user to create a new mana source button with colors that they select, and have the app display cards appropriately.

This problem can be broken into three parts:

1. Select colors and create a new mana source button.
2. Allow custom multicolor mana sources to be used for any of their defined colors.
3. Allow custom multicolor mana sources to be used only as many times as the user indicates are available (ie. a source that creates blue or black should only count for 1 mana and not 2).

### Select colors and create a new mana source button

The easiest part of this problem was the UI. I knew I wanted to have some kind of circular menu that presented the 5 colors where the user could select 2-4 colors, and then confirm or cancel.

I looked at a few different open source libraries for creating circular menus, and I landed on [react-planet](https://github.com/innFactory/react-planet) as it was more flexible and visually pretty much what I wanted.

I created a state object within my multicolorMenu component called colors, and listed the 5 main colors that I'd need to make buttons for.

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

multicolormenu.png

<br>

A few things happen when one of the buttons is clicked. First, the selected value for that color is changed to true so that I can conditionally display a checkmark over the button indicating that it's selected. Second, the color object is added to a `colorsForNewManaSource` array in state, and then the array is sorted by each object's `sortOrder` value. This is important later on.

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

Input video here

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

Now when a new source is created, the colors value for that object is created by using the `split()` method on the source's name. So the new source added to the mana state looks like this:

```js
WU: { value: 0, colors: ['W', 'U']},
```

This required a change to the function that compares casting cost to available mana. Before, I was able to loop over the mana and directly compare each source to the required color since it was just a numerical value. Now I had to check for each source if it produced the current color it was checking for, and only compare the available amount if it did produce that color.

### Allow a multicolor source to be used only as many times as are available as indicated by the user

I ran into an issue where multicolor sources were allowing cards that cost more mana than was available through the filter. For example, I had 1 mana from a source providing blue or red, and a card costing 1 blue _and_ 1 red was being displayed. After a lot of troubleshooting, including learning a lot about how to use the debugger in the browser, I discovered the issue. When the function checked for the availability of multiple colors, it did not indicate if a source had already been used, so it was using the full available mana for each color of a card. So using the above example, when the function checked if there was 1 blue mana, it found that there was. Then when it moved on to red, it found that there was also 1 red mana available, because there was no indication that it had already been used to provide the required blue mana.

So I needed a solution. I also wanted to prioritize using sources that produced fewer colors since they are less flexible. If I had 1 blue from an only blue source and 1 blue/red mana, and the same card costing 1 blue and 1 red, I didn't want to inadvertantly use up the blue/red mana on the required blue mana and have an extra blue mana and no red mana available.

I solved both issues at once. I made a temporary copy of the current mana state as an array that I could manipulate throughout the checking of an individual card without affecting the actual mana. Next, the array was by the number of colors each source could produce. In this way, the order was guaranteed to be the same every time, and I'd avoid any issues using the mana inefficiently.

Time to change the compare function one more time. Now I loop over each color of required mana, and for each required color loop over the mana state.

```javascript
for (currentColor in requiredMana) {
  mana.forEach((source) => {
    // End the loop if the source has no mana available, or if the required color has been fulfilled.
    if (source.value === 0 || requiredMana[currentColor] === 0) return;

    if (source.colors.includes(currentColor)) {
      /*  If there is more mana available than required for the current color,
       subtract the required amount from the mana source. Otherwise, subtract the
       available mana from the required amount. */
      if (source.value >= requiredMana[currentColor]) {
        source.value -= requiredMana[currentColor];
        if (source.value < 0) {
          source.value = 0;
        }
        color[1] = 0;
      } else {
        requiredMana[currentColor] -= source.value;
        if (requiredMana[currentColor] < 0) {
          requiredMana[currentColor] = 0;
        }
        requiredMana[currentColor] = 0;
      }
    }
  });
}
```

<br>

Now I have a menu that allows the user to create their own multicolor mana buttons. The buttons can be used the same as the standard ones, and properly display any cards that match the colors and amount of mana provided by those sources. It's probably not the most efficient solution, though I'll probably never consider this project done. I'll probably come back to it and improve it along my journey, but for where I'm at in my career right now, I'm happy with the fact that it works.

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

# Today I learned what getStaticProps and getServerSideProps do

I've been using React for a couple years, and Next.js for a little over a year though for much more straightforward projects.

I'm used to using the `useState` and `useEffect` hooks in React. Normally, I'd set my state variables in a side effect, and then use render the page using information from the state variables. So imagine my frustration when I was trying to do the same thing in Next only to get constant errors about my state variables being undefined!

I was trying to display a table using data from a json file. I initially supplied that data to a state variable:

```js
import { survivors as survivorsList } from '../components/survivors';
import { players as playersList } from '../components/players';

export default function Home({ playersList, survivorsList }) {
  const [players, setPlayers] = useState([]);
  const [survivors, setSurvivors] = useState([]);

  const getData = () => {
    setPlayers(playersList);
    setSurvivors(survivorsList);
  };

  useEffect(() => {
    getData();
  }, [playersList, survivorsList]);

```

Then I created the table rows by looping over the players array and using the appropriate data. I was rewarded with `TypeError: can't access property "map", players is undefined`.

What was happening without my understanding was that Next was pre-rendering the page. On the initial render my state variables were indeed undefined, and after the render my side effect fired and populated them with the data. The problem is that when the page relies on some of that data to render the pre-render that Next was doing fails, providing the error message that my variable was undefined.

A quick google search into why Next was rendering before my side effect led me to the page of the Next.js docs about Pages and Pre-rendering. What I learned was that unlike React which runs the `useEffect` before rendering, Next generates HTML for each page in advance for performance and SEO reasons. In order to pre-render a page that relies on some data, you have to pass the data to the page as props.

You do this using one of two functions: `getStaticProps` or `getServerSideProps`.

`getStaticProps` is used for static generation. Next generates the HTML at build time, and this HTML is supplied for every request. `getServerSideProps` is used for server-side rendering. Here, Next generates HTML on each request. This is useful if the page shows frequently updated data.

After playing around with the functions I managed to get my page to render using the supplied data, using `getStaticProps`. Now I know when I eventually start using data that I'll have to fetch from a server, I'll have to change it to `getServerSideProps` in order to pre-render the page with the current data.

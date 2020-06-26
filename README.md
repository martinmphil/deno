# Deno script
Mathematical modelling to create json data for polyhedral probabilities.

## Installation
Install `deno` by following the instructions at [deno.land](https://deno.land/).

Then, in the project directory, run
`deno run --allow-write create_json_data.ts`
to create the data file called
`data.json`.

## TypeScript code extract
```ts
interface I {
  n: number;
  d: number;
  t: number;
  f: number;
}

const nArr: number[] = [1, 2, 3, 4, 5, 6, 7, 8, 9];
const polyhedral: number[] = [6, 8, 12, 20, 100];

const targetArr = (d: number) => {
  return Array.from({ length: d - 1 }, (_, i) => i + 1);
};

const forceRatio = (prob: number) => {
  let f: number;
  if (prob > 0.995) {
    f = 100;
  } else if (prob > 0.75) {
    f = 0.5 * (1 / (1 - prob));
  } else if (prob > 0.5) {
    f = 4 * prob - 1;
  } else {
    f = 2 * prob;
  }
  return parseFloat(f.toPrecision(4));
};

const encounter = (n: number, d: number, t: number): I => {
  const prob = 1 - (1 - t / d) ** n;
  const f = forceRatio(parseFloat(prob.toPrecision(4)));
  return { n, d, t, f };
};

const partyArr = (n: number, polyhedral: number[]): I[] => {
  const everyCombo = polyhedral.flatMap((d) => {
    return targetArr(d).map((t) => {
      return encounter(n, d, t);
    });
  });
  const results = everyCombo.filter((x, i, arr) => {
    return arr.map((y) => y.f).indexOf(x.f) === i;
  });
  return results;
};

const orunderData: I[] = nArr.flatMap((n) => {
  return partyArr(n, polyhedral);
});

const encoder = new TextEncoder();
await Deno.writeFile("data.json", encoder.encode(JSON.stringify(orunderData)));
```

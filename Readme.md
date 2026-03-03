```js
const GIST_URL = 'https://gist.githubusercontent.com/kmdupr33/8bab92e762d367de9455183abe04f38f/raw/';

async function fetchArt(url) {
    const res = await fetch(url);
    const text = await res.text();
    const blocks = [...text.matchAll(/```[\s\S]*?```/g)];
    if (!blocks.length) throw new Error('No code blocks found in gist');
    return blocks[blocks.length - 1][0]
        .replace(/^```[^\n]*\n?/, '')
        .replace(/```$/, '');
}

function countPupilChars(line) {
    const matches = line.match(/[^ ()'`\-,]/g);
    return matches ? matches.length : 0;
}

function findEyesLine(lines) {
    return lines.find(line => {
        const trimmed = line.trim();
        return trimmed.startsWith('(') && trimmed.includes(')') && countPupilChars(trimmed) > 0;
    });
}

function findLipsLine(lines) {
    for (let i = 15; i < lines.length; i++) {
        const line = lines[i];
        const trimmed = line.trim();
        if (trimmed.length > 2 && !/['`,-]/.test(line)) {
            return { line, index: i };
        }
    }
    return null;
}

function countPolkadots(lines, lipsStart, lipsEnd) {
    let inside = 0, outside = 0;
    for (const line of lines) {
        for (let x = 0; x < line.length; x++) {
            if (line[x] === 'O') {
                if (lipsStart !== -1 && x >= lipsStart && x <= lipsEnd) inside++;
                else outside++;
            }
        }
    }
    return { inside, outside };
}

async function computePolkadotScore() {
    const art = await fetchArt(GIST_URL);
    const lines = art.replace(/\r\n/g, '\n').split('\n');

    const eyesLine = findEyesLine(lines);
    if (!eyesLine) throw new Error('Could not find eyes line');
    const pupilChars = countPupilChars(eyesLine);

    const lips = findLipsLine(lines);
    const lipsTrimmed = lips ? lips.line.trim() : '';
    const lipsStart = lips ? lips.line.indexOf(lipsTrimmed) : -1;
    const lipsEnd = lips ? lipsStart + lipsTrimmed.length - 1 : -1;

    const { inside, outside } = countPolkadots(lines, lipsStart, lipsEnd);
    const score = outside + inside * pupilChars;

    console.log(`Eyes line:    "${eyesLine.trim()}"`);
    console.log(`Pupils:       ${pupilChars}`);
    console.log(`Lips line:    "${lips ? lips.line : 'NOT FOUND'}"`);
    console.log(`Lips X-Range: ${lipsStart} to ${lipsEnd}`);
    console.log(`Inside O:     ${inside}`);
    console.log(`Outside O:    ${outside}`);
    console.log(`Score:        ${outside} + (${inside} * ${pupilChars}) = ${score}`);

    return score;
}

computePolkadotScore();
```

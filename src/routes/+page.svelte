<script lang="ts">
	const maxNumber = 12;
	let schriftlicheNoten = $state(Array(maxNumber));
	let muendlicheNoten = $state(Array(maxNumber));

	let gewichtungen = $derived.by(() => {
		let gewichtungen: (null | 1 | 2)[] = muendlicheNoten.map((note) => (note ? 1 : null));
		let indizesNochZuGewichten = muendlicheNoten
			.map((note, index) => index)
			.filter((index) => muendlicheNoten[index]);

		while (indizesNochZuGewichten.length) {
			let aktuellerSchnitt = schnittBerechnen(schriftlicheNoten, muendlicheNoten, gewichtungen);
			let notenNochZuGewichten: Array<number> = indizesNochZuGewichten.map(
				(index) => muendlicheNoten[index]
			);
			let besteNote = Math.min(...notenNochZuGewichten);
			let indexDerBestenNote = muendlicheNoten.findIndex(
				(el, i) => el === besteNote && indizesNochZuGewichten.includes(i)
			);

			if (besteNote < aktuellerSchnitt) {
				gewichtungen[indexDerBestenNote] = 2;
				indizesNochZuGewichten = indizesNochZuGewichten.filter((i) => i !== indexDerBestenNote);
			} else {
				break;
			}
		}
		return gewichtungen;
	});

	let schnitt = $derived(schnittBerechnen(schriftlicheNoten, muendlicheNoten, gewichtungen));
	$inspect(muendlicheNoten, gewichtungen);

	function schnittBerechnen(
		schriftlicheNoten: number[],
		muendlicheNoten: number[],
		gewichtungen: (null | 1 | 2)[]
	) {
		let summe = schriftlicheNoten.reduce((prev, cur) => prev + cur, 0);
		let divisor = schriftlicheNoten.filter((number) => number).length;

		summe += muendlicheNoten.reduce((prev, cur, i) => prev + cur * Number(gewichtungen[i]), 0);
		divisor += gewichtungen.reduce((prev, cur) => prev + Number(cur), 0);

		return summe / divisor;
	}

	function gewichtungText(gewichtung: null | 1 | 2) {
		if (gewichtung == 2) return 'doppelt';
		if (gewichtung == 1) return 'einfach';
		return '';
	}
</script>

<h1>WHG Günstigerprüfung LRS</h1>

<small>Schulaufgabennoten nicht eingeben!</small>

<div class="schriftlicheNoten">
	Schriftliche kleine Leistungsnachweise: <br />
	{#each schriftlicheNoten as note, i}
		<div class="schriftlicheNote">
			<input type="number" min="1" max="6" step="1" bind:value={schriftlicheNoten[i]} />
		</div>
	{/each}
</div>

<div class="muendlicheNoten">
	Mündliche kleine Leistungsnachweise:<br />
	<table>
		<tbody>
			<tr>
				{#each muendlicheNoten as note, i}
					<td><input type="number" min="1" max="6" step="1" bind:value={muendlicheNoten[i]} /></td>
				{/each}
			</tr>
			<tr class="gewichtungen">
				{#each gewichtungen as gewichtung}
					<td class={gewichtung == 2 ? 'doppelt' : ''}
						><div>{gewichtung} <br /> <small>{gewichtungText(gewichtung)}</small></div></td
					>
				{/each}
			</tr>
		</tbody>
	</table>
</div>

<p>
	Schnitt der kleinen Leistungnachweise: <b>
		{schnitt.toLocaleString(undefined, {
			minimumFractionDigits: 2,
			maximumFractionDigits: 2
		})}
	</b>
</p>

<style>
	* {
		font-family: sans-serif;
	}
	td {
		padding: 0;
		text-align: center;
	}

	td.doppelt div {
		background-color: orange;
	}

	table {
		border-collapse: collapse;
	}

	div.schriftlicheNoten,
	div.muendlicheNoten {
		padding: 20px 0;
	}

	tr.gewichtungen td div {
		padding: 5px;
		margin: 3px 5px;
	}

	input {
		padding: 5px;
		margin: 5px;
	}

	div.schriftlicheNote {
		display: inline-block;
	}
</style>

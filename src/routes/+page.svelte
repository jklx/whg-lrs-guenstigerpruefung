<script lang="ts">
	const maxNumber = 12;
	let oberstufe = $state(false);
	let schriftlicheNotenEingabe = $state(Array(maxNumber));
	let muendlicheNotenEingabe = $state(Array(maxNumber));
	let schriftlicheNoten = $derived(
		schriftlicheNotenEingabe.map((note) => eingabeAuwerten(note)).filter((note) => note !== undefined)
	);
	let muendlicheNoten = $derived(
		muendlicheNotenEingabe.map((note) => eingabeAuwerten(note)).filter((note) => note !== undefined)
	);

	let gewichtungen = $derived.by(() => {
		let gewichtungen: (null | 1 | 2)[] = muendlicheNoten.map(() => 1);
		let indizesNochZuGewichten = muendlicheNoten
			.map((note, index) => index);

		while (indizesNochZuGewichten.length) {
			let aktuellerSchnitt = schnittBerechnen(schriftlicheNoten, muendlicheNoten, gewichtungen);
			let notenNochZuGewichten: Array<number> = indizesNochZuGewichten.map(
				(index) => muendlicheNoten[index]
			);
			let besteNote = findeBesteNote(notenNochZuGewichten);
			let indexDerBestenNote = muendlicheNoten.findIndex(
				(el, i) => el === besteNote && indizesNochZuGewichten.includes(i)
			);

			if (istNoteBesserAlsSchnitt(besteNote, aktuellerSchnitt)) {
				gewichtungen[indexDerBestenNote] = 2;
				indizesNochZuGewichten = indizesNochZuGewichten.filter((i) => i !== indexDerBestenNote);
			} else {
				break;
			}
		}
		return gewichtungen;
	});

	let schnitt = $derived(schnittBerechnen(schriftlicheNoten, muendlicheNoten, gewichtungen));
	let originalSchnitt = $derived(schnittBerechnen(schriftlicheNoten, muendlicheNoten));

	function findeBesteNote(noten: number[]) {
		if (oberstufe) {
			return Math.max(...noten);
		} else {
			return Math.min(...noten);
		}
	}

	function istNoteBesserAlsSchnitt(note: number, schnitt: number) {
		if (oberstufe) {
			return note > schnitt;
		} else {
			return note < schnitt;
		}
	}

	function schnittBerechnen(
		schriftlicheNoten: number[],
		muendlicheNoten: number[],
		gewichtungen?: (null | 1 | 2)[]
	) {
		let summe = schriftlicheNoten.reduce((prev, cur) => prev + cur, 0);
		let divisor = schriftlicheNoten.length;

		if (gewichtungen !== undefined) {
			summe += muendlicheNoten.reduce((prev, cur, i) => prev + cur * Number(gewichtungen[i]), 0);
			divisor += gewichtungen.reduce((prev, cur) => prev + Number(cur), 0);
		} else {
			summe += muendlicheNoten.reduce((prev, cur) => prev + cur, 0);
			divisor += muendlicheNoten.length;
		}

		return summe / divisor;
	}

	function eingabeAuwerten(note: string) { 
		note = note.replace(/^[\s+-]+|[\s+-]+$/g, ''); // Entfernt führende und nachfolgende Leerzeichen und Plus-/Minuszeichen
		if (note.match(/^\d+$/)) {
			return Number(note);
		} else return undefined; // Gibt null zurück, wenn die Eingabe ungültig ist
	}

	function gewichtungText(gewichtung: null | 1 | 2) {
		if (gewichtung == 2) return 'doppelt';
		if (gewichtung == 1) return 'einfach';
		return '';
	}

	function reset() {
		schriftlicheNotenEingabe = Array(maxNumber);
		muendlicheNotenEingabe = Array(maxNumber);
	}
</script>

<h1>WHG Günstigerprüfung LRS</h1>


<div class="notensystem">
	<label>
		<input type="radio" bind:group={oberstufe} value={false} />
		Notensystem 1-6
	</label>
	<label>
		<input type="radio" bind:group={oberstufe} value={true} />
		Punktesystem 0-15
	</label>
</div>

<small>Schulaufgabennoten nicht eingeben!</small> <br />


<div class="schriftlicheNoten">
	Schriftliche kleine Leistungsnachweise: (Exen, Tests, ...) <br />
	{#each schriftlicheNotenEingabe as note, i}
		<div class="schriftlicheNote">
			<input type="text" size="2" bind:value={schriftlicheNotenEingabe[i]} />
		</div>
	{/each}
</div>

<div class="muendlicheNoten">
	Mündliche kleine Leistungsnachweise: (Abfragen, UB-Noten, ...)<br />
	<table>
		<tbody>
			<tr>
				{#each muendlicheNotenEingabe as note, i}
					<td><input type="text" size="2" bind:value={muendlicheNotenEingabe[i]} /></td>
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
	<span class="schnittLabel">Schnitt der kleinen Leistungnachweise ohne Doppeltgewichtung:</span>
	{originalSchnitt.toLocaleString(undefined, {
		minimumFractionDigits: 2,
		maximumFractionDigits: 2,
		roundingMode: 'floor'
	})}
	<br />
	<span class="schnittLabel">Schnitt der kleinen Leistungnachweise mit Günstigerprüfung:</span>
	<b>
		{schnitt.toLocaleString(undefined, {
			minimumFractionDigits: 2,
			maximumFractionDigits: 2,
			roundingMode: 'floor'
		})}
	</b>
</p>

<p>
	<button onclick={reset}>Zurücksetzen</button>
</p>

<style>
	* {
		font-family: sans-serif;
	}

	div.notensystem {
		margin: 15px 0;
	}

	div.notensystem label {
		margin-right: 20px;
	}

	td {
		padding: 0;
		text-align: center;
	}

	tr.gewichtungen td {
		height: 53px;
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

	span.schnittLabel {
		display: inline-block;
		width: 500px;
	}
</style>

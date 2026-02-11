<script>
	import { onMount } from 'svelte';
	import { pipeline, env } from '@huggingface/transformers';

	// Yerel model kullanımını kapat (Modeli Hugging Face Hub'dan çeker)
	env.allowLocalModels = false;

	// --- Değişkenler ---
	let classifier = null; // Modeli tutacak değişken
	let status = 'Model yükleniyor...'; // Durum mesajı
	let inputText = 'I love creating apps with Svelte!'; // Varsayılan metin
	let result = null; // Analiz sonucu
	let isLoading = true; // Yükleniyor mu?

	// --- 1. Uygulama açılınca Modeli Yükle ---
	onMount(async () => {
		try {
			// 'sentiment-analysis' pipeline'ı varsayılan olarak
			// 'distilbert-base-uncased-finetuned-sst-2-english' modelini indirir.
			classifier = await pipeline('sentiment-analysis');

			isLoading = false;
			status = 'Model hazır!';

			// İlk açılışta varsayılan metni analiz et
			await analyze();
		} catch (err) {
			status = 'Hata: ' + err.message;
			isLoading = false;
		}
	});

	// --- 2. Analiz Fonksiyonu ---
	async function analyze() {
		if (!classifier || !inputText) return;

		// Modeli çalıştır
		const output = await classifier(inputText);

		// Çıktı formatı şöyledir: [{ label: 'POSITIVE', score: 0.99... }]
		result = output[0];
	}

	// --- 3. Otomatik Tetikleme (Reactivity) ---
	// inputText her değiştiğinde bu blok çalışır.
	// Performans için buraya 'debounce' eklenebilir ama basitlik adına doğrudan bağladık.
	$: if (classifier && inputText) {
		analyze();
	}
</script>

<main class="mx-auto max-w-2xl px-4 py-12 font-sans">
	{#if isLoading}
		<div class="mb-4 badge animate-pulse rounded badge-info">
			⏳ {status} (İlk yükleme biraz zaman alabilir)
		</div>
	{:else}
		<div class="mb-4 badge rounded badge-success">
			✅ {status}
		</div>
	{/if}

	<div class="mb-6">
		<label for="sentiment-input" class="label mb-2 block">
			Analiz edilecek İngilizce metni girin:
		</label>
		<textarea
			id="sentiment-input"
			bind:value={inputText}
			disabled={isLoading}
			rows="4"
			class="textarea w-full rounded transition"
			placeholder="Buraya bir şeyler yazın..."
		></textarea>
	</div>

	{#if result}
		<div class="card">
			<h2 class="card-title">Sonuç</h2>

			<div class="card-body">
				<div class="flex justify-between">
					<div>
						<span>Duygu:</span>
						<span
							class:text-success={result.label === 'POSITIVE'}
							class:text-error={result.label === 'NEGATIVE'}
							class="font-bold"
						>
							{result.label}
						</span>
					</div>

					<div>
						<span>Güven Skoru:</span>
						<span class="font-mono">
							%{(result.score * 100).toFixed(2)}
						</span>
					</div>
				</div>
			</div>

			<progress
				class:progress-success={result.label === 'POSITIVE'}
				class:progress-error={result.label === 'NEGATIVE'}
				class="progress w-full transition-all duration-500"
				value={result.score * 100}
				max="100"
			></progress>
		</div>
	{/if}
</main>

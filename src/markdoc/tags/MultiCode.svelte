<script context="module" lang="ts">
    import type { Writable } from 'svelte/store';
    export type CodeContext = {
        selected: Writable<string | null>;
        snippets: Writable<Set<Language>>;
        content: Writable<string>;
    };
</script>

<script lang="ts">
    import { platformMap } from '$lib/utils/references';
    import { getContext, setContext } from 'svelte';
    import { writable } from 'svelte/store';
    import type { Language } from '$lib/utils/code';
    import { copy } from '$lib/utils/copy';
    import { Select, Tooltip } from '$lib/components';

    setContext<CodeContext>('multi-code', {
        selected: writable(null),
        snippets: writable(new Set()),
        content: writable('')
    });

    const languageContext = getContext<Writable<string>>('language-context');

    const { snippets, selected, content } = getContext<CodeContext>('multi-code');

    snippets.subscribe((n) => {
        if ($selected === null && n.size > 0) {
            $selected = Array.from(n)[0];
        }
    });

    selected.subscribe((language) => {
        // apply if exists in snippets
        if (language && $snippets.has(language as Language)) {
            languageContext?.set(language);
        }
    });

    languageContext?.subscribe((language) => {
        if (
            language &&
            language !== $selected &&
            // apply if exists in snippets
            $snippets.has(language as Language)
        ) {
            selected.set(language);
        }
    });

    enum CopyStatus {
        Copy = 'Copy',
        Copied = 'Copied!'
    }
    let copyText = CopyStatus.Copy;
    async function handleCopy() {
        await copy($content);

        copyText = CopyStatus.Copied;
        setTimeout(() => {
            copyText = CopyStatus.Copy;
        }, 1000);
    }
</script>

<section class="dark web-code-snippet" aria-label="code-snippet panel">
    <header class="web-code-snippet-header">
        <div class="web-code-snippet-header-start">
            <div class="flex gap-4">
                <!-- <div class="web-tag"><span class="text">Default</span></div> -->
            </div>
        </div>
        <div class="web-code-snippet-header-end">
            <ul class="buttons-list flex gap-3">
                <li class="buttons-list-item flex self-center">
                    <Select
                        bind:value={$selected}
                        options={Array.from($snippets).map((language) => ({
                            value: language,
                            label: platformMap[language]
                        }))}
                    />
                </li>

                <li class="buttons-list-item" style="padding-inline-start: 13px">
                    <Tooltip>
                        <button
                            on:click={handleCopy}
                            class="web-icon-button"
                            aria-label="copy code from code-snippet"
                            ><span class="web-icon-copy" aria-hidden="true" /></button
                        >
                        <svelte:fragment slot="tooltip">
                            {copyText}
                        </svelte:fragment>
                    </Tooltip>
                </li>
            </ul>
        </div>
    </header>
    <div class="web-code-snippet-content"><slot /></div>
</section>

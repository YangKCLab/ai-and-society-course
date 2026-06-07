<script setup>
import { computed } from "vue";
import { marked } from "marked";
import { useVersion } from "../composables/useVersion";

const { currentVersion } = useVersion();

// Load all essay markdown files at build time via glob
// Each key looks like: ../../../versions/Spring2026/content/essays/essay-1.md
const allFiles = import.meta.glob("../../../versions/*/content/essays/*.md", {
    query: "?raw",
    import: "default",
    eager: true,
});

/**
 * Parse simple YAML frontmatter from a raw markdown string.
 * Returns { frontmatter: { title, author, order }, body }.
 * Frontmatter values are plain strings; order is coerced to a number.
 */
function parseFrontmatter(raw) {
    const match = raw.match(/^---\r?\n([\s\S]*?)\r?\n---\r?\n?([\s\S]*)$/);
    if (!match) return { frontmatter: {}, body: raw };

    const fm = {};
    for (const line of match[1].split("\n")) {
        const sep = line.indexOf(":");
        if (sep < 1) continue;
        const key = line.slice(0, sep).trim();
        const val = line.slice(sep + 1).trim();
        fm[key] = val;
    }
    if (fm.order !== undefined) fm.order = Number(fm.order);

    return { frontmatter: fm, body: match[2] };
}

// Build the essay list for the current version, sorted by order then filename
const essays = computed(() => {
    const version = currentVersion.value;
    const list = [];

    for (const [key, raw] of Object.entries(allFiles)) {
        // key pattern: ../../../versions/{Version}/content/essays/{slug}.md
        const segments = key.split("/");
        // Find the index of 'versions' in the path
        const vIdx = segments.findIndex((s) => s === "versions");
        if (vIdx < 0) continue;
        const fileVersion = segments[vIdx + 1];
        if (fileVersion !== version) continue;

        const filename = segments[segments.length - 1]; // e.g. "essay-1.md"
        const slug = filename.replace(/\.md$/, "");
        const { frontmatter, body } = parseFrontmatter(raw);

        list.push({
            id: slug,
            title: frontmatter.title || "",
            author: frontmatter.author || "",
            order:
                typeof frontmatter.order === "number"
                    ? frontmatter.order
                    : Infinity,
            html: marked.parse(body),
        });
    }

    return list.sort((a, b) => {
        if (a.order !== b.order) return a.order - b.order;
        return a.id.localeCompare(b.id);
    });
});
</script>

<template>
    <main class="container pb-5">
        <h1 class="h3 my-4">Student Essays</h1>

        <p>
            One of the assignments required the students to write an essay
            reflecting on how AI had changed their lives and work by the end of
            the semester. Some students gave permission for their essays to be
            shared publicly. You can read them below.
        </p>

        <p v-if="!essays.length" class="text-muted">
            Student essays will be posted soon.
        </p>

        <template v-else>
            <!-- Essay cards (collapsed by default) -->
            <div v-for="essay in essays" :key="essay.id" class="card mb-3">
                <details class="essay-card">
                    <summary class="card-body essay-summary">
                        <span :id="essay.id" class="h6 mb-0">{{
                            essay.author || essay.title || essay.id
                        }}</span>
                        <span
                            v-if="essay.title && essay.author"
                            class="text-muted small ms-2"
                            >— {{ essay.title }}</span
                        >
                    </summary>
                    <div class="card-body pt-0">
                        <!-- eslint-disable-next-line vue/no-v-html -->
                        <div class="essay-body" v-html="essay.html"></div>
                    </div>
                </details>
            </div>
        </template>
    </main>
</template>

<style scoped>
main {
    max-width: 860px;
}

.essay-summary {
    cursor: pointer;
    display: flex;
    align-items: baseline;
    gap: 0.25rem;
    list-style: none; /* hide default triangle on some browsers */
    user-select: none;
}

/* Restore disclosure triangle via pseudo-element */
.essay-summary::before {
    content: "▶";
    font-size: 0.65rem;
    color: #6c757d;
    margin-right: 0.4rem;
    transition: transform 0.15s ease;
    flex-shrink: 0;
}

.essay-card[open] > .essay-summary::before {
    transform: rotate(90deg);
}

.essay-body :deep(p) {
    margin-bottom: 0.75rem;
}

.essay-body :deep(h1),
.essay-body :deep(h2),
.essay-body :deep(h3) {
    font-size: 1rem;
    font-weight: 600;
    margin-top: 1.25rem;
    margin-bottom: 0.5rem;
}
</style>

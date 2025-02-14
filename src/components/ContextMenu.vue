<template>
    <NDropdown trigger="manual" :show="lyricLineMenu.show" @select="onSelectMenu" :x="lyricLineMenu.x" :y="lyricLineMenu.y"
        placement="bottom-start" :options="lyricLineMenu.selectedWord === -1 ? lineContextMenu : wordOnlyContextMenu"
        @clickoutside="lyricLineMenu.show = false" />
</template>

<script setup lang="ts">
import {
    NDropdown,
    useNotification,
} from "naive-ui";
import { onMounted } from "vue";
import { useRightClickLyricLine, useEditingLyric } from "../store";
import type { DropdownMixedOption } from "naive-ui/es/dropdown/src/interface";
import { i18n } from '../i18n';

const lineContextMenu = [
    { label: i18n.global.t("contextMenu.deleteLine"), key: 'delete-line' },
    { label: i18n.global.t("contextMenu.insertBeforeLine"), key: 'insert-before-line' },
    { label: i18n.global.t("contextMenu.insertAfterLine"), key: 'insert-after-line' },
    { label: i18n.global.t("contextMenu.toggleBGLine"), key: 'toggle-bg-line' },
    { label: i18n.global.t("contextMenu.toggleDuetLine"), key: 'toggle-duet-line' },
] as DropdownMixedOption[];
const wordOnlyContextMenu = [
    { label: i18n.global.t("contextMenu.deleteWord"), key: 'delete-word' },
    { label: i18n.global.t("contextMenu.splitWord"), key: 'split-word' },
    { type: 'divider', },
    ...lineContextMenu
] as DropdownMixedOption[];

const lyricLineMenu = useRightClickLyricLine();
const notify = useNotification();
const lyric = useEditingLyric();

function onSelectMenu(key: string) {
    switch (key) {
        case "delete-line": {
            lyric.removeLine(lyricLineMenu.selectedLine);
            break;
        }
        case "delete-word": {
            lyric.removeWord(lyricLineMenu.selectedLine, lyricLineMenu.selectedWord);
            break;
        }
        case "toggle-bg-line": {
            const line = lyric.lyrics[lyricLineMenu.selectedLine];
            if (line) line.isBG = !line.isBG;
            break;
        }
        case "toggle-duet-line": {
            const line = lyric.lyrics[lyricLineMenu.selectedLine];
            if (line) line.isDuet = !line.isDuet;
            break;
        }
        case "insert-before-line": {
            lyric.insertNewLineAt(lyricLineMenu.selectedLine);
            break;
        }
        case "insert-after-line": {
            lyric.insertNewLineAt(lyricLineMenu.selectedLine + 1);
            break;
        }
        default: {
            notify.error({
                title: i18n.global.t("contextMenu.wipNotification.title"),
                content: i18n.global.t("contextMenu.wipNotification.content"),
                duration: 4000,
            });
        }
    }
    lyricLineMenu.show = false;
}

onMounted(() => {
    // ask before page close
    window.addEventListener("beforeunload", evt => {
        evt.preventDefault();
        return evt.returnValue = "";
    })
});
</script>
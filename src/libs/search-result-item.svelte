<script lang="ts">
    import { showMessage } from "siyuan";
    import { DocumentSearchResultItem } from "@/libs/search-data";
    import { getBlockTypeIconHref } from "@/libs/icons";

    // import { onMount } from "svelte";

    export let searchResults: DocumentSearchResultItem[];
    export let clickCallback: (item: DocumentSearchResultItem) => void;
    export let selectedIndex: number = 0;

    // $: renderedSearchResultItems = searchResults.slice(0, 10);

    function itemClick(item: DocumentSearchResultItem) {
        if (clickCallback) {
            clickCallback(item);
        }
    }

    function handleKeyDownDefault(event) {
        console.log(event.key);
    }

    function toggleItemVisibility(block: Block) {
        if (!block || !block.id) {
            return;
        }
        for (const item of searchResults) {
            if (!item || !item.block) {
                continue;
            }
            if (item.block.id === block.id) {
                if (item.subItems && item.subItems.length > 0) {
                    item.isCollapsed = !item.isCollapsed;
                    searchResults = searchResults;
                } else {
                    showMessage(
                        `${item.block.content} 下不存在符合条件的内容`,
                        5000,
                        "error",
                    );
                }
                break;
            }
        }
    }

    // function loadMore() {
    //     // 加载更多数据
    //     const currentIndex = renderedSearchResultItems.length;
    //     const nextBatch = searchResults.slice(currentIndex, currentIndex + 10);
    //     renderedSearchResultItems = [
    //         ...renderedSearchResultItems,
    //         ...nextBatch,
    //     ];
    // }
</script>

<div
    id="searchList"
    class="fn__flex-1 search__list b3-list b3-list--background"
>
    {#each searchResults as item}
        <!-- on:click={() => itemClick(item.block)} -->
        <div
            class="b3-list-item {item.index === selectedIndex
                ? 'b3-list-item--focus'
                : ''} "
            on:click|stopPropagation={() => toggleItemVisibility(item.block)}
            on:keydown={handleKeyDownDefault}
            data-node-id={item.block.id}
            data-root-id={item.block.root_id}
        >
            <!-- 按钮折叠改为移动到整个文档标签。                on:click|stopPropagation={() =>
                    toggleItemVisibility(item.block)} -->
            <span
                class="b3-list-item__toggle b3-list-item__toggle--hl
                {item.subItems && item.subItems.length > 0 ? '' : 'disabled'}
                "
            >
                <svg
                    class="b3-list-item__arrow
                    {item.isCollapsed ? '' : 'b3-list-item__arrow--open'}
                    {item.subItems && item.subItems.length > 0
                        ? ''
                        : 'disabled'}"
                >
                    <use xlink:href="#iconRight"></use>
                </svg>
            </span>
            <span class="b3-list-item__graphic">
                {#if item.icon}
                    {@html item.icon}
                {:else}
                    📄
                {/if}
            </span>
            <span
                class="b3-list-item__text ariaLabel"
                style="color: var(--b3-theme-on-surface)"
                aria-label={item.path}
            >
                {@html item.block.content}
            </span>
            <div class="protyle-attr--refcount" style="right:0px;top:6px">
                {item.subItems.length}
            </div>
        </div>
        <div>
            {#each item.subItems as subItem (subItem.block.id)}
                <div
                    style="padding-left: 36px"
                    data-type="search-item"
                    class="b3-list-item {item.isCollapsed ? 'fn__none' : ''}
                {subItem.index === selectedIndex ? 'b3-list-item--focus' : ''}"
                    data-node-id={subItem.block.id}
                    data-root-id={subItem.block.root_id}
                    on:click={() => itemClick(subItem)}
                    on:keydown={handleKeyDownDefault}
                >
                    <svg class="b3-list-item__graphic">
                        <use
                            xlink:href={getBlockTypeIconHref(
                                subItem.block.type,
                                subItem.block.subtype,
                            )}
                        ></use>
                    </svg>
                    <span class="b3-list-item__text"
                        >{@html subItem.block.content}
                    </span>

                    {#if subItem.block.name}
                        <span
                            class="b3-list-item__meta fn__flex"
                            style="max-width: 30%"
                        >
                            <svg class="b3-list-item__hinticon">
                                <use xlink:href="#iconN"></use>
                            </svg><span class="b3-list-item__hinttext">
                                {@html subItem.block.name}
                            </span>
                        </span>
                    {/if}
                    {#if subItem.block.alias}
                        <span
                            class="b3-list-item__meta fn__flex"
                            style="max-width: 30%"
                        >
                            <svg class="b3-list-item__hinticon">
                                <use xlink:href="#iconA"></use>
                            </svg><span class="b3-list-item__hinttext">
                                {@html subItem.block.alias}
                            </span>
                        </span>
                    {/if}
                    {#if subItem.block.memo}
                        <span
                            class="b3-list-item__meta fn__flex"
                            style="max-width: 30%"
                        >
                            <svg class="b3-list-item__hinticon"
                                ><use xlink:href="#iconM"></use></svg
                            ><span class="b3-list-item__hinttext">
                                {@html subItem.block.memo}
                            </span>
                        </span>
                    {/if}
                </div>
            {/each}
        </div>
    {/each}
</div>

<style style="CSS">
    .disabled {
        /* pointer-events: none; */ /* 禁止元素接受用户的鼠标事件 */
        opacity: 0.5; /* 设置元素透明度，表示禁用状态 */
        /* cursor: not-allowed;*/ /* 改变鼠标光标，表示不允许交互 */
    }
</style>

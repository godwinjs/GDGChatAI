<script lang="ts">
  import { ChatBubble } from "$lib/ui";

  let files: FileList;
  let question = "";
  let conversation: { id: number; text: string; isQuestion: boolean }[] = [];
  let isLoading = false;
  let error: string | null = null;

  async function ask() {
    isLoading = true;
    const newQuestion = {
      id: conversation.length + 1,
      text: question,
      isQuestion: true,
    };
    conversation = [...conversation, newQuestion];

    try {
      const formData = new FormData();

      // Append all files
      for (let i = 0; i < files.length; i++) {
        formData.append("files", files[i]);
      }

      formData.append("question", question);

      const response = await fetch("/api/file", {
        method: "POST",
        body: formData,
      });

      if (response.ok) {
        const data = await response.json();
        question = "";
        const newResponse = {
          id: conversation.length + 1,
          text: data,
          isQuestion: false,
        };
        conversation = [...conversation, newResponse];
      } else {
        error = "Failed to fetch data. Please try again.";
      }
    } catch (e) {
      error = "An error occurred. Please try again later.";
    } finally {
      question = "";
      isLoading = false;
    }
  }
</script>

<div class="h-full w-full overflow-y-auto justify-center items-center relative">
  <div class="grid grid-row-[1fr_auto] w-full mb-24">
    {#if conversation.length === 0}
      <ChatBubble
        item={{
          id: 1,
          text: "Hi! I am NodeBot. Ask me anything and I will try to answer it.",
          isQuestion: false,
        }}
      />
    {/if}
    {#each conversation as item (item.id)}
      <ChatBubble {item} />
    {/each}
  </div>
  {#if isLoading}
    <ChatBubble isLoading={true} />
  {/if}
  {#if error}
    <p>{error}</p>
  {/if}
  <div
    class="border-t border-surface-500/30 bg-surface-800 p-4 fixed bottom-0 w-[-webkit-fill-available] h-18 overflow-x-none"
  >
    <div
      class="input-group input-group-divider grid-cols-[auto_1fr_auto] rounded-container-token"
    >
      <!-- Allow multiple PDF and CSV files -->
      <input
        class="bg-transparent border-0 ring-0 mt-1 ml-1"
        title="File Input"
        type="file"
        name="files"
        accept=".pdf, .csv, .docx, .txt"
        multiple
        bind:files
        disabled={isLoading}
      />
      <input
        class="bg-transparent border-0 ring-0"
        title="Question Input"
        type="text"
        name="question"
        placeholder="Ask your question here..."
        bind:value={question}
        disabled={isLoading}
      />
      <button type="button" class="input-group-shim" on:click={ask}
        >Submit</button
      >
    </div>
  </div>
</div>

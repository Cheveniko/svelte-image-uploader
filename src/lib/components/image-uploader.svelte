<script lang="ts">
  import * as Form from "$lib/components/ui/form";
  import Dropzone from "svelte-file-dropzone";
  import { fileProxy, superForm } from "sveltekit-superforms";
  import { zodClient } from "sveltekit-superforms/adapters";
  import { toast } from "svelte-sonner";
  import { cn } from "$lib/utils";
  import { z } from "zod";
  import { ImagePlus } from "lucide-svelte";
  import Spinner from "$lib/components/spinner.svelte";

  let disabled = $state(false);
  let preview = $state<string | ArrayBuffer | null>("");
  let inputRef = $state<HTMLInputElement | null>(null);
  let isDragActive = $state(false);

  const formSchema = z.object({
    image: z
      .instanceof(File, { message: "Please upload an image" })
      .refine((f) => f.size < 5_000_000, "The image should be less than 5MB")
      .refine(
        (f) => f.type === "image/jpeg" || f.type === "image/png",
        "Only jpg, jpeg or png files are accepted",
      ),
  });

  const form = superForm(
    { image: new File([""], "filename") },
    {
      validators: zodClient(formSchema),
      SPA: true,
      onUpdate: ({ form }) => {
        if (form.valid) {
          console.log(form);
          preview = null;
          toast.success(`Image uploaded successfully 🎉 ${form.data.image.name}`);
        }
      },
    },
  );

  let file = $state(fileProxy(form, "image"));

  let files: { accepted: File[]; rejected: File[] } = $state({
    accepted: [],
    rejected: [],
  });

  function onDrop(e: { detail: { acceptedFiles: File[]; fileRejections: File[] } }) {
    const { acceptedFiles, fileRejections } = e.detail;
    files.accepted = acceptedFiles;
    files.rejected = fileRejections;

    const reader = new FileReader();
    try {
      reader.onload = () => (preview = reader.result);
      reader.readAsDataURL(acceptedFiles[0]);
      $formData.image = acceptedFiles[0];
    } catch {
      preview = null;
      $formData.image = new File([""], "filename");
    }
  }

  function oninput(e: Event & { currentTarget: EventTarget & HTMLInputElement }) {
    const reader = new FileReader();
    const acceptedFile = e.currentTarget.files?.[0];

    if (!acceptedFile) {
      preview = null;
      $formData.image = new File([""], "filename");
      return;
    }

    files.rejected = [];

    try {
      reader.onload = () => (preview = reader.result);
      reader.readAsDataURL(acceptedFile);
      $formData.image = acceptedFile;
    } catch {
      preview = null;
      $formData.image = new File([""], "filename");
    }
  }

  const { form: formData, enhance } = form;
</script>

<form method="POST" enctype="multipart/form-data" use:enhance class="space-y-4">
  <Form.Field {form} name="image" class="mx-auto md:w-1/2">
    <Form.Control>
      {#snippet children({ props })}
        <Form.Label
          class={cn(
            "text-xl font-semibold tracking-tight",
            files.rejected.length !== 0 && "text-destructive",
          )}
        >
          Upload your image
        </Form.Label>
        <Dropzone
          inputElement={inputRef}
          accept={["image/jpeg", "image/png"]}
          maxSize={5_000_000}
          on:dragenter={() => (isDragActive = true)}
          on:dragleave={() => (isDragActive = false)}
          on:drop={(e) => {
            onDrop(e);
            isDragActive = false;
          }}
          class="border-foreground shadow-foreground mx-auto flex cursor-pointer flex-col items-center justify-center gap-y-2 rounded-lg border p-8 shadow-sm"
        >
          {#if preview}
            <img src={preview as string} alt="" class="max-h-[400px] rounded-lg" />
          {:else}
            <ImagePlus class="size-40" />
          {/if}
          <input
            {...props}
            bind:this={inputRef}
            bind:files={$file}
            {oninput}
            type="file"
            class="hidden"
          />
          {#if isDragActive}
            <p>Drop the image!</p>
          {:else}
            <p>Click here or drag an image to upload it</p>
          {/if}
        </Dropzone>
      {/snippet}
    </Form.Control>
    <Form.FieldErrors />
    {#if files.rejected.length !== 0}
      <Form.FieldErrors>
        The image must be less than 5MB and of type png, jpg or jpeg
      </Form.FieldErrors>
    {/if}
  </Form.Field>
  <Form.Button {disabled} class="mx-auto block h-auto rounded-lg px-8 py-3 text-xl">
    {#if disabled}
      <Spinner class="mx-auto animate-spin" />
    {:else}
      Submit
    {/if}
  </Form.Button>
</form>

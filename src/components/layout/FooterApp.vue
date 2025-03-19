<template>
  <footer
    class="absolute left-0 mt-10 w-full min-h-[200px] text-lg flex flex-col justify-center items-center h-16 bg-slate-500/20 border-t border-slate-500/50"
  >
    <div class="flex p-4 divide-x divide-slate-500/80">
      <RouterLink to="/about" class="px-4 font-thin underline underline-offset-2">About</RouterLink>
      <RouterLink to="/about#legal-mentions" class="px-4 font-thin underline underline-offset-2"
        >Legal Mentions</RouterLink
      >
      <a
        href="https://github.com/FacuBotta/metadata-pdf-editor"
        class="px-4 font-thin underline underline-offset-2"
        target="_blank"
        >GitHub Repo</a
      >
    </div>
    <p>
      Made with ❤️ & ⌨️ by
      <a class="underline underline-offset-2" href="https://facubotta.netlify.app/" target="_blank"
        >Facu Botta</a
      >
    </p>
  </footer>
</template>

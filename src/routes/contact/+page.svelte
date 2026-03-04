<svelte:head>
  <title>Contact — Sri Saraf</title>
</svelte:head>
<div class="page-inner">
  <h1>Contact</h1>
  <form action="mailto:srisaraf@mit.edu" method="GET">
   <label for="email">Email</label>
   <input id="email" type="email" name="email" value="srisaraf@mit.edu">
   <label for="subject">Subject</label>
   <input id="subject" name="subject" value="hello">
   <label for="msg">Message</label>
   <textarea id="msg" name="body">hi</textarea>
   <button>Submit</button>
  </form>
</div>
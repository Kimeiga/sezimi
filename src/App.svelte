<script>
  import { FirebaseApp, User, Doc, Collection } from "sveltefire";

  import firebase from "firebase/app";
  import "firebase/firestore";
  import "firebase/auth";
  import "firebase/performance";
  import "firebase/analytics";

  // For Firebase JS SDK v7.20.0 and later, measurementId is optional
  const firebaseConfig = {
    apiKey: "AIzaSyAtDKR5I37ekCTnU7qvABzzD6Oj4QqbOFc",
    authDomain: "sezimi.firebaseapp.com",
    projectId: "sezimi",
    storageBucket: "sezimi.appspot.com",
    messagingSenderId: "892819239644",
    appId: "1:892819239644:web:39a8dd2c786fb49d50fa19",
    measurementId: "G-H15MK1Y25N",
  };

  firebase.initializeApp(firebaseConfig);
  const db = firebase.firestore();

  const googleProvider = new firebase.auth.GoogleAuthProvider();

  let postText;
</script>

<main>
  <!-- 1. 🔥 Firebase App -->
  <FirebaseApp {firebase}>
    <h1>sezimi</h1>

    <!-- 2. 😀 Get the current user -->
    <User let:user let:auth>
      <button on:click={() => auth.signOut()}>Sign Out</button>

      <div slot="signed-out">
        <p>
          what if twitter but instead of 250 character limit you only had one
          character 0_0
        </p>
        <button
          on:click={() => {
            auth.signInWithPopup(googleProvider).then((result) => {
              db.collection("users")
                .doc(result.user.uid)
                .set({
                  displayName: result.user.displayName,
                  photoURL: result.user.photoURL,
                  createdAt: firebase.firestore.FieldValue.serverTimestamp(),
                })
                .then(() => {
                  console.log("User document successfully written!");
                })
                .catch((error) => {
                  console.error("Error writing user document: ", error);
                });
            });
          }}
        >
          Sign In with Google
        </button>
      </div>

      <Collection
        path={"posts/"}
        query={(ref) => ref.orderBy("createdAt")}
        let:data={posts}
        let:ref={postsRef}
        log
      >
        <div
          class="composer card"
          style="display: flex; flex-direction: column;"
        >
          <div class="flex" style="display: flex;">
            <img
              src={user.photoURL}
              alt="user's face idk"
              class="profile-picture"
            />

            <textarea
              bind:value={postText}
              style="width: 100%"
              placeholder="Wha happen?"
            />
          </div>
          <button
            style="align-self:flex-end;"
            on:click={() =>
              postsRef.add({
                text: postText,
                createdAt: Date.now(),
                uid: user.uid,
                likes: 0,
              })}
          >
            Twet
          </button>
        </div>

        {#each posts as post}
          <Doc path={`users/${post.uid}`} let:data={user}>
            <div class="card">
              <div style="display: flex;">
                <img
                  src={user.photoURL}
                  alt="user's face idk"
                  class="profile-picture"
                />
                <div style="display: flex; flex-direction: column;">
                  <div>
                    <strong>{user.displayName}</strong>
                    <small>@{post.uid}</small>
                  </div>
                  <h1>{post.text}</h1>
                  <div style="display: flex; justify-content: space-between;">
                    <button
                      on:click={() => {
                        postsRef.doc(post.id).update({
                          likes: firebase.firestore.FieldValue.increment(1),
                        });
                      }}>&lt;3 {post.likes}</button
                    >
                    <button on:click={() => post.ref.delete()}>Delete</button>
                  </div>
                </div>
              </div>
            </div>
          </Doc>
        {/each}
      </Collection>
    </User>
  </FirebaseApp>
</main>

<style>
  :global(body) {
    background: black;
    color: white;
  }
  main {
    max-width: 50rem;
    margin: auto;
    /* border: white 1px solid; */
  }

  .card {
    width: 100%;
    padding: 1rem;
    border: white 1px solid;
  }
  .profile-picture {
    width: 50px;
    margin-right: 10px;
    border-radius: 100px;
    align-self: flex-start;
  }
</style>

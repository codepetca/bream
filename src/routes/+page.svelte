<script lang="ts">
  import { onMount } from 'svelte';

  // 6 pairs, 12 cards
  const images = [
    'https://api.dicebear.com/9.x/fun-emoji/svg?seed=cat',
    'https://api.dicebear.com/9.x/fun-emoji/svg?seed=dog',
    'https://api.dicebear.com/9.x/fun-emoji/svg?seed=elephant',
    'https://api.dicebear.com/9.x/fun-emoji/svg?seed=lion',
    'https://api.dicebear.com/9.x/fun-emoji/svg?seed=tiger',
    'https://api.dicebear.com/9.x/fun-emoji/svg?seed=bear'
  ];

  type Card = {
    id: number;
    image: string;
    matched: boolean;
    flipped: boolean;
  };

  let cards: Card[] = [];
  let firstCard: Card | null = null;
  let secondCard: Card | null = null;
  let lockBoard = false;
  let moves = 0;
  let matches = 0;
  let gameWon = false;

  function shuffle<T>(array: T[]): T[] {
    let arr = array.slice();
    for (let i = arr.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
    return arr;
  }

  function resetGame() {
    const doubled = images.concat(images);
    cards = shuffle(
      doubled.map((img, i) => ({
        id: i,
        image: img,
        matched: false,
        flipped: false
      }))
    );
    firstCard = null;
    secondCard = null;
    lockBoard = false;
    moves = 0;
    matches = 0;
    gameWon = false;
  }

  function flipCard(card: Card) {
    if (lockBoard || card.flipped || card.matched) return;
    card.flipped = true;
    cards = [...cards];
    if (!firstCard) {
      firstCard = card;
    } else if (!secondCard) {
      secondCard = card;
      moves++;
      if (firstCard.image === secondCard.image) {
        firstCard.matched = true;
        secondCard.matched = true;
        cards = [...cards];
        matches++;
        if (matches === images.length) {
          gameWon = true;
        }
        firstCard = null;
        secondCard = null;
      } else {
        lockBoard = true;
        setTimeout(() => {
          if (firstCard) firstCard.flipped = false;
          if (secondCard) secondCard.flipped = false;
          cards = [...cards];
          firstCard = null;
          secondCard = null;
          lockBoard = false;
        }, 900);
      }
    }
  }

  onMount(resetGame);
</script>

<style>
.game-container {
  max-width: 400px;
  margin: 0 auto;
  padding: 1rem;
}
.grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 0.7rem;
}
.card {
  aspect-ratio: 1/1;
  background: #eee;
  border-radius: 0.7rem;
  box-shadow: 0 2px 8px #0001;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
  touch-action: manipulation;
  cursor: pointer;
  user-select: none;
  transition: transform 0.1s;
}
.card.flipped, .card.matched {
  box-shadow: 0 4px 16px #0002;
  background: #fff;
}
.card img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 0.7rem;
  pointer-events: none;
}
.card .back {
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #4a90e2 0%, #a7c7e7 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2rem;
  color: #fff;
  border-radius: 0.7rem;
  pointer-events: none;
}
.win-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  cursor: pointer;
}
.win-message {
  background: white;
  padding: 2rem 3rem;
  border-radius: 1rem;
  font-size: 2rem;
  text-align: center;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}
button {
  display: block;
  margin: 1.2rem auto 0 auto;
  padding: 0.7rem 1.5rem;
  font-size: 1.1rem;
  border: none;
  border-radius: 0.5rem;
  background: #a7c7e7;
  color: #222;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.2s;
}
button:hover {
  background: #f7cac9;
}
@media (max-width: 500px) {
  .game-container {
    max-width: 98vw;
    padding: 0.5rem;
  }
  .grid {
    gap: 0.4rem;
  }
}
</style>

<div class="game-container">
  {#if gameWon}
    <div class="win-overlay" on:click={resetGame} on:keydown={(e) => e.key === 'Enter' && resetGame()} role="button" tabindex="0">
      <div class="win-message">🎉 You won!</div>
    </div>
  {/if}
  <div class="grid">
    {#each cards as card (card.id)}
      <div
        class="card {card.flipped || card.matched ? 'flipped' : ''} {card.matched ? 'matched' : ''}"
        on:click={() => flipCard(card)}
        on:keydown={(e) => e.key === 'Enter' && flipCard(card)}
        aria-label={card.flipped || card.matched ? 'Flipped card' : 'Hidden card'}
        role="button"
        tabindex="0"
      >
        {#if card.flipped || card.matched}
          <img src={card.image} alt="" />
        {:else}
          <div class="back">🐾</div>
        {/if}
      </div>
    {/each}
  </div>
</div>

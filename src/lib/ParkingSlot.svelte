<script>
  let { id, status, isSelected, onClick } = $props();

  // Status can be 'available', 'occupied', 'banned'
</script>

<button 
  class="slot {status} {isSelected ? 'selected' : ''}" 
  onclick={onClick}
  disabled={status === 'occupied'}
  aria-label={`Parking Spot ${id} ${status}`}
>
  <div class="slot-content">
    <span class="slot-id">{id}</span>
    {#if status === 'occupied'}
      <span class="icon">🚗</span>
    {/if}
  </div>
</button>

<style>
  .slot {
    width: 100%;
    aspect-ratio: 1/1.2;
    border-radius: var(--radius-md);
    background-color: var(--color-surface);
    border: 2px solid transparent;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    position: relative;
    overflow: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 60px;
    padding: 8px;
  }

  @media (max-width: 768px) {
    .slot {
      min-height: 50px;
      padding: 6px;
    }
  }

  @media (max-width: 480px) {
    .slot {
      min-height: 45px;
      padding: 4px;
    }
  }

  .slot.available {
    border-color: rgba(255,255,255,0.05);
  }
  
  .slot.available:hover {
    background-color: var(--color-surface-hover);
    transform: translateY(-2px);
    border-color: var(--color-primary);
    box-shadow: 0 0 15px var(--color-primary-glow);
  }

  .slot.selected {
    background-color: rgba(245, 158, 11, 0.1);
    border-color: var(--color-selected);
    box-shadow: 0 0 20px rgba(245, 158, 11, 0.3);
    z-index: 10;
  }

  .slot.occupied {
    background-color: rgba(239, 68, 68, 0.1);
    border-color: rgba(239, 68, 68, 0.3);
    opacity: 0.7;
    cursor: not-allowed;
  }

  .slot-content {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
  }

  .slot-id {
    font-size: 0.85rem;
    font-weight: 700;
    color: var(--color-text-muted);
    line-height: 1;
    text-align: center;
  }

  .selected .slot-id {
    color: var(--color-selected);
  }

  .icon {
    font-size: 1rem;
    line-height: 1;
  }

  @media (max-width: 768px) {
    .slot-id {
      font-size: 0.75rem;
    }
    .icon {
      font-size: 0.9rem;
    }
  }

  @media (max-width: 480px) {
    .slot-id {
      font-size: 0.7rem;
    }
    .icon {
      font-size: 0.8rem;
    }
  }
</style>

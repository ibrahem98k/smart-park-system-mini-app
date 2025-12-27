<script lang="ts">
    import { onMount } from 'svelte';
    import ParkingSlot from "./ParkingSlot.svelte";

    interface Spot {
        id: string;
        status: 'available' | 'occupied' | 'banned';
    }

    export let spots: Spot[] = [];
    export let selectedSpotId: string | null = null;
    export let onSelect: (id: string) => void = () => {};

    let isMounted = false;
    
    onMount(() => {
        isMounted = true;
        return () => {
            isMounted = false;
        };
    });
</script>

<div class="parking-lot">
    <div class="parking-rows">
        <div class="row">
            {#each spots.slice(0, 6) as spot, i}
                <div class="slot-wrapper" style="--delay: {i * 0.05}s; animation-delay: {i * 0.05}s">
                    <ParkingSlot
                        id={spot.id}
                        status={spot.status}
                        isSelected={selectedSpotId === spot.id}
                        onClick={() => onSelect(spot.id)}
                    />
                </div>
            {/each}
        </div>

        <div class="road">
            <div class="road-content">
                <div class="entry-indicator">
                    <div class="arrow">↓</div>
                    <span class="entry-text">ENTRY</span>
                </div>
            </div>
        </div>

        <div class="row">
            {#each spots.slice(6, 12) as spot, i}
                <div class="slot-wrapper" style="--delay: ${(i + 6) * 0.05}s; animation-delay: ${(i + 6) * 0.05}s">
                    <ParkingSlot
                        id={spot.id}
                        status={spot.status}
                        isSelected={selectedSpotId === spot.id}
                        onClick={() => onSelect(spot.id)}
                    />
                </div>
            {/each}
        </div>
    </div>
    
    <div class="parking-lot-footer">
        <div class="parking-lot-info">
            <div class="info-item">
                <span class="info-label">Available</span>
                <span class="info-value">{spots.filter(s => s.status === 'available').length}</span>
            </div>
            <div class="info-item">
                <span class="info-label">Total</span>
                <span class="info-value">{spots.length}</span>
            </div>
        </div>
    </div>
</div>

<style>
    .parking-lot {
        background: rgba(30, 41, 59, 0.8);
        backdrop-filter: blur(10px);
        border-radius: var(--radius-lg);
        box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
        overflow: hidden;
        border: 1px solid rgba(255, 255, 255, 0.08);
        transition: all 0.3s ease;
    }

    .parking-rows {
        padding: 20px;
    }

    .row {
        display: grid;
        grid-template-columns: repeat(6, 1fr);
        gap: 10px;
        margin-bottom: 15px;
    }

    @media (max-width: 768px) {
        .row {
          grid-template-columns: repeat(4, 1fr);
          gap: 8px;
        }
    }

    @media (max-width: 480px) {
        .row {
          grid-template-columns: repeat(3, 1fr);
          gap: 6px;
        }
    }

    .road {
        display: flex;
        align-items: center;
        justify-content: center;
        margin: 20px 0;
        position: relative;
        height: 40px;
    }

    .road::before {
        content: '';
        position: absolute;
        top: 50%;
        left: 0;
        right: 0;
        height: 1px;
        background: linear-gradient(90deg, 
            transparent 0%, 
            rgba(99, 102, 241, 0.6) 50%, 
            transparent 100%);
        transform: translateY(-50%);
    }

    .road-content {
        position: relative;
        z-index: 1;
        display: flex;
        align-items: center;
        justify-content: center;
        width: 100%;
    }

    .entry-indicator {
        display: flex;
        align-items: center;
        gap: 8px;
        background: rgba(99, 102, 241, 0.1);
        padding: 6px 16px;
        border-radius: 20px;
        border: 1px solid rgba(99, 102, 241, 0.3);
        backdrop-filter: blur(10px);
    }

    .arrow {
        font-size: 1rem;
        color: var(--color-primary);
        animation: bounce 2s infinite;
    }

    .entry-text {
        font-size: 0.7rem;
        font-weight: 600;
        color: var(--color-primary);
        letter-spacing: 1px;
        text-transform: uppercase;
    }

    @keyframes bounce {
        0%, 100% {
            transform: translateY(0);
        }
        50% {
            transform: translateY(3px);
        }
    }

    .parking-lot-footer {
        background: rgba(30, 41, 59, 0.3);
        padding: 12px 16px;
        border-top: 1px solid rgba(255, 255, 255, 0.08);
        backdrop-filter: blur(10px);
    }

    .parking-lot-info {
        display: flex;
        justify-content: space-around;
        align-items: center;
        gap: 20px;
    }

    .info-item {
        display: flex;
        align-items: center;
        gap: 8px;
        background: rgba(255, 255, 255, 0.05);
        padding: 8px 16px;
        border-radius: 12px;
        border: 1px solid rgba(255, 255, 255, 0.1);
        transition: all 0.3s ease;
    }

    .info-item:hover {
        background: rgba(255, 255, 255, 0.08);
        border-color: rgba(255, 255, 255, 0.15);
        transform: translateY(-1px);
    }

    .info-label {
        font-size: 0.7rem;
        color: rgba(255, 255, 255, 0.6);
        text-transform: uppercase;
        letter-spacing: 0.8px;
        font-weight: 500;
    }

    .info-value {
        font-size: 1.1rem;
        font-weight: 700;
        color: var(--color-primary);
        text-shadow: 0 0 8px rgba(99, 102, 241, 0.3);
    }

    @media (max-width: 768px) {
        .parking-lot-footer {
            padding: 10px 12px;
        }
        
        .parking-lot-info {
            gap: 15px;
        }
        
        .info-item {
            padding: 6px 12px;
        }
        
        .info-label {
            font-size: 0.65rem;
        }
        
        .info-value {
            font-size: 1rem;
        }
    }

    @media (max-width: 480px) {
        .parking-lot-footer {
            padding: 8px 10px;
        }
        
        .parking-lot-info {
            gap: 10px;
        }
        
        .info-item {
            padding: 5px 10px;
            flex-direction: column;
            gap: 4px;
        }
        
        .info-label {
            font-size: 0.6rem;
        }
        
        .info-value {
            font-size: 0.9rem;
        }
    }
</style>

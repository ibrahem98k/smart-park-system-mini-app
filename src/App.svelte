<script module lang="ts">
  // Declare the Alipay Mini Program global object at module level
  declare const my: {
    getAuthCode: (options: {
      scopes: string[];
      success: (res: { authCode: string }) => void;
      fail: (res: any) => void;
    }) => void;
    alert: (options: { content: string }) => void;
    tradePay: (options: {
      paymentUrl: string;
      success: (res: any) => void;
    }) => void;
    scan: (options: {
      type: string;
      success: (res: { code: string }) => void;
    }) => void;
  } | undefined;
</script>

<script lang="ts">
  import { onMount } from 'svelte';
  import ParkingLot from "./lib/ParkingLot.svelte";
  import BookingModal from "./lib/BookingModal.svelte";
  import Ticket from "./lib/Ticket.svelte";

  type SpotStatus = 'available' | 'occupied' | 'banned';
  
  interface Spot {
    id: string;
    status: SpotStatus;
  }

  interface BookingTicket {
    spotId: string | null;
    duration: number;
  }

  // Different parking data for each level
  const levelData: Record<number, Spot[]> = {
    1: [
      { id: "A1", status: "available" as const },
      { id: "A2", status: "occupied" as const },
      { id: "A3", status: "available" as const },
      { id: "A4", status: "available" as const },
      { id: "A5", status: "banned" as const },
      { id: "A6", status: "available" as const },
      { id: "B1", status: "available" as const },
      { id: "B2", status: "available" as const },
      { id: "B3", status: "occupied" as const },
      { id: "B4", status: "occupied" as const },
      { id: "B5", status: "available" as const },
      { id: "B6", status: "available" as const },
    ],
    2: [
      { id: "C1", status: "available" as const },
      { id: "C2", status: "available" as const },
      { id: "C3", status: "occupied" as const },
      { id: "C4", status: "available" as const },
      { id: "C5", status: "available" as const },
      { id: "C6", status: "banned" as const },
      { id: "D1", status: "occupied" as const },
      { id: "D2", status: "available" as const },
      { id: "D3", status: "available" as const },
      { id: "D4", status: "occupied" as const },
      { id: "D5", status: "available" as const },
      { id: "D6", status: "available" as const },
    ],
    3: [
      { id: "E1", status: "available" as const },
      { id: "E2", status: "occupied" as const },
      { id: "E3", status: "available" as const },
      { id: "E4", status: "banned" as const },
      { id: "E5", status: "available" as const },
      { id: "E6", status: "available" as const },
      { id: "F1", status: "available" as const },
      { id: "F2", status: "occupied" as const },
      { id: "F3", status: "available" as const },
      { id: "F4", status: "available" as const },
      { id: "F5", status: "occupied" as const },
      { id: "F6", status: "available" as const },
    ]
  };

  // Reactive state with enhanced data
  let spots = $state<Spot[]>(levelData[1]);

  let selectedSpotId = $state<string | null>(null);
  let showModal = $state<boolean>(false);
  let currentTicket = $state<BookingTicket | null>(null);
  let authCode = $state<string>("");
  let token = $state<string>("");
  let currentLevel = $state<number>(1);
  let isLoading = $state<boolean>(false);
  let notification = $state<{message: string, type: 'success' | 'error' | 'info'} | null>(null);

  function handleSelect(id: string): void {
    const spot = spots.find((s) => s.id === id);
    if (!spot || spot.status !== "available") {
      showNotification('This spot is not available', 'error');
      return;
    }

    selectedSpotId = id;
    showModal = true;
  }

  function handleLevelChange(level: number): void {
    currentLevel = level;
    selectedSpotId = null;
    spots = levelData[level];
    showNotification(`Switched to Level ${level}`, 'info');
  }

  function showNotification(message: string, type: 'success' | 'error' | 'info'): void {
    notification = { message, type };
    setTimeout(() => {
      notification = null;
    }, 3000);
  }

  function handleConfirm(duration: number): void {
    if (!selectedSpotId) return;

    // Logic: Authenticate -> Pay -> Show Ticket
    // For now, we assume user is authenticated at mount or we verify it here
    if (!token) {
      authenticate().then(() => {
        pay(duration);
      });
    } else {
      pay(duration);
    }
  }

  function authenticate(): Promise<void> {
    return new Promise<void>((resolve, reject) => {
      if (typeof my === "undefined") {
        console.warn("MiniApp environment not detected");
        resolve(); // Mock success for web dev
        return;
      }

      my.getAuthCode({
        scopes: ["auth_base", "USER_ID"],
        success: (res) => {
          authCode = res.authCode;

          fetch("https://its.mouamle.space/api/auth-with-superQi", {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              token: authCode,
            }),
          })
            .then((res) => res.json())
            .then((data) => {
              token = data.token;
              my.alert({ content: "Login successful" });
              resolve();
            })
            .catch((err) => {
              let errorDetails = "";
              if (err && typeof err === "object") {
                errorDetails = JSON.stringify(err, null, 2);
              } else {
                errorDetails = String(err);
              }
              my.alert({ content: "Error: " + errorDetails });
              reject(err);
            });
        },
        fail: (res) => {
          console.log(res.authErrorScopes);
          reject(res);
        },
      });
    });
  }

  function pay(duration: number): void {
    if (typeof my === "undefined") {
      // Mock payment for web
      completeBooking(duration);
      return;
    }

    fetch("https://its.mouamle.space/api/payment", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: token,
      },
    })
      .then((res) => res.json())
      .then((data) => {
        my.tradePay({
          paymentUrl: data.url,
          success: (res) => {
            my.alert({
              content: "Payment successful",
            });
            completeBooking(duration);
          },
        });
      })
      .catch((err) => {
        my.alert({
          content: "Payment failed",
        });
      });
  }

  function completeBooking(duration: number): void {
    // Update spot status
    const idx = spots.findIndex((s) => s.id === selectedSpotId);
    if (idx !== -1) {
      spots[idx].status = "occupied";
    }

    currentTicket = { spotId: selectedSpotId, duration };

    // Close modal
    showModal = false;
    selectedSpotId = null;
  }

  function handleCloseTicket(): void {
    currentTicket = null;
  }

  function handleScan(): void {
    if (typeof my === "undefined") {
      // Mock functionality for browser testing
      console.log("Simulating QR Scan");
      selectedSpotId = "MALL-PARKING-A1";
      showModal = true;
      return;
    }
    my.scan({
      type: "qr",
      success: (res) => {
        // Use the scanned code (Park Name/ID) to initiate booking
        if (res.code) {
          selectedSpotId = res.code;
          showModal = true;
        }
      },
    });
  }

  onMount(() => {
    // Optional: Try to silent login on mount
    authenticate();
  });
</script>

<main class="app-container">
  <header class="app-header">
    <div class="top-bar">
      <button class="icon-button" aria-label="Go back">
        <span class="back-arrow">←</span>
      </button>
      <h1>Parking Lot</h1>
      <button class="scan-btn" onclick={handleScan} aria-label="Scan QR Code">
        <span>📷</span>
      </button>
      <div class="user-avatar"></div>
    </div>

    <div class="status-summary">
      <div class="status-pill status-free">
        <span class="dot"></span> Available
      </div>
      <div class="status-pill status-busy">
        <span class="dot"></span> Occupied
      </div>
    </div>
  </header>

  <div class="content-area">
    <div class="floor-selector">
      <button 
        class={currentLevel === 1 ? 'active' : ''} 
        onclick={() => handleLevelChange(1)}
      >
        Level 1
      </button>
      <button 
        class={currentLevel === 2 ? 'active' : ''} 
        onclick={() => handleLevelChange(2)}
      >
        Level 2
      </button>
      <button 
        class={currentLevel === 3 ? 'active' : ''} 
        onclick={() => handleLevelChange(3)}
      >
        Level 3
      </button>
    </div>

    <ParkingLot {spots} {selectedSpotId} onSelect={handleSelect} />
  </div>

  {#if showModal}
    <BookingModal
      spotId={selectedSpotId}
      onCancel={() => {
        showModal = false;
        selectedSpotId = null;
      }}
      onConfirm={handleConfirm}
    />
  {/if}

  {#if currentTicket}
    <Ticket
      spotId={currentTicket.spotId}
      duration={currentTicket.duration}
      onClose={handleCloseTicket}
    />
  {/if}
</main>

<style>
  header {
    padding: 20px 24px;
    background: linear-gradient(
      180deg,
      rgba(30, 41, 59, 1) 0%,
      rgba(15, 23, 42, 0) 100%
    );
  }

  @media (max-width: 768px) {
    header {
      padding: 15px 16px;
    }
  }

  @media (max-width: 480px) {
    header {
      padding: 10px 12px;
    }
  }

  .top-bar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
  }

  h1 {
    font-size: 1.2rem;
    margin: 0;
    background: linear-gradient(90deg, #fff 0%, #a5b4fc 100%);
    background-clip: text;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    letter-spacing: -0.5px;
  }

  @media (max-width: 768px) {
    h1 {
      font-size: 1.1rem;
    }
  }

  @media (max-width: 480px) {
    h1 {
      font-size: 1rem;
    }
  }

  .back-arrow {
    font-size: 1.5rem;
    line-height: 1;
    display: inline-block;
    transition: transform 0.2s;
  }

  .icon-button:hover .back-arrow {
    transform: translateX(-2px);
  }

  .icon-button {
    background: rgba(255, 255, 255, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.2);
    border-radius: 50%;
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: all 0.3s ease;
    backdrop-filter: blur(10px);
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  }

  .scan-btn {
    background: linear-gradient(135deg, rgba(99, 102, 241, 0.2), rgba(168, 85, 247, 0.2));
    border: 1px solid rgba(99, 102, 241, 0.3);
    border-radius: 50%;
    width: 36px;
    height: 36px;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-right: 10px;
    margin-left: auto;
    cursor: pointer;
    transition: all 0.3s ease;
    backdrop-filter: blur(10px);
    box-shadow: 0 4px 15px rgba(99, 102, 241, 0.2);
  }

  .scan-btn:hover {
    transform: scale(1.05);
    box-shadow: 0 6px 20px rgba(99, 102, 241, 0.3);
    border-color: rgba(99, 102, 241, 0.5);
  }

  .user-avatar {
    width: 36px;
    height: 36px;
    background: linear-gradient(135deg, #6366f1, #a855f7);
    border-radius: 50%;
    box-shadow: 0 0 10px rgba(168, 85, 247, 0.4);
    position: relative;
    overflow: hidden;
    cursor: pointer;
    transition: all 0.3s ease;
  }

  .user-avatar::before {
    content: '';
    position: absolute;
    top: 2px;
    left: 2px;
    right: 2px;
    bottom: 2px;
    background: linear-gradient(135deg, rgba(255, 255, 255, 0.2), transparent);
    border-radius: 50%;
  }

  .user-avatar:hover {
    transform: scale(1.05);
    box-shadow: 0 0 15px rgba(168, 85, 247, 0.6);
  }

  .status-summary {
    display: flex;
    gap: 15px;
    justify-content: center;
  }

  .status-pill {
    display: flex;
    align-items: center;
    gap: 6px;
    font-size: 0.8rem;
    color: var(--color-text-muted);
    background: rgba(255, 255, 255, 0.05);
    padding: 6px 12px;
    border-radius: 20px;
    border: 1px solid rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    transition: all 0.3s ease;
  }

  .status-pill:hover {
    background: rgba(255, 255, 255, 0.08);
    border-color: rgba(255, 255, 255, 0.15);
    transform: translateY(-1px);
  }

  .dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
  }

  .status-free .dot {
    background: var(--color-primary);
    box-shadow: 0 0 8px var(--color-primary);
  }
  .status-busy .dot {
    background: var(--color-booked);
  }

  .content-area {
    padding: 20px;
  }

  @media (max-width: 768px) {
    .content-area {
      padding: 15px;
    }
  }

  @media (max-width: 480px) {
    .content-area {
      padding: 10px;
    }
  }

  .floor-selector {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
    overflow-x: auto;
    padding-bottom: 5px;
  }

  .floor-selector button {
    padding: 8px 16px;
    border-radius: 20px;
    background: rgba(255, 255, 255, 0.05);
    color: var(--color-text-muted);
    font-size: 0.9rem;
    font-weight: 500;
    white-space: nowrap;
    border: 1px solid rgba(255, 255, 255, 0.1);
    cursor: pointer;
    transition: all 0.3s ease;
    backdrop-filter: blur(10px);
  }

  .floor-selector button:hover:not(.active) {
    background: rgba(255, 255, 255, 0.1);
    border-color: rgba(255, 255, 255, 0.2);
    transform: translateY(-1px);
  }

  @media (max-width: 768px) {
    .floor-selector button {
      padding: 6px 12px;
      font-size: 0.85rem;
    }
  }

  @media (max-width: 480px) {
    .floor-selector button {
      padding: 5px 10px;
      font-size: 0.8rem;
    }
  }

  .floor-selector button.active {
    background: var(--color-primary);
    color: #fff;
    box-shadow: 0 4px 12px var(--color-primary-glow);
  }
</style>

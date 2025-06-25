import React, { useState, useEffect, useCallback } from 'react';

const InvitationGame = () => {
  // 게임 상태 관리
  const [gameState, setGameState] = useState('playing'); // 'playing', 'party', 'finished'
  const [playerPosition, setPlayerPosition] = useState({ x: 50, y: 50 });
  const [invitedNPCs, setInvitedNPCs] = useState(new Set());
  const [resolution, setResolution] = useState('');
  const [showMessage, setShowMessage] = useState('');
  const [messageTimeout, setMessageTimeout] = useState(null);

  // NPC 데이터
  const npcs = [
    { id: 'parents', name: '부모님', x: 150, y: 100, emoji: '👨‍👩‍👧‍👦' },
    { id: 'friend', name: '친구', x: 300, y: 200, emoji: '👫' },
    { id: 'teacher', name: '선생님', x: 500, y: 150, emoji: '👨‍🏫' },
    { id: 'colleague', name: '동기', x: 400, y: 300, emoji: '👥' }
  ];

  // 키보드 이벤트 처리
  const handleKeyPress = useCallback((e) => {
    if (gameState !== 'playing') return;

    const moveSpeed = 10;
    const gameArea = { width: 600, height: 400 };

    switch (e.key) {
      case 'ArrowUp':
        e.preventDefault();
        setPlayerPosition(prev => ({ 
          ...prev, 
          y: Math.max(0, prev.y - moveSpeed) 
        }));
        break;
      case 'ArrowDown':
        e.preventDefault();
        setPlayerPosition(prev => ({ 
          ...prev, 
          y: Math.min(gameArea.height - 40, prev.y + moveSpeed) 
        }));
        break;
      case 'ArrowLeft':
        e.preventDefault();
        setPlayerPosition(prev => ({ 
          ...prev, 
          x: Math.max(0, prev.x - moveSpeed) 
        }));
        break;
      case 'ArrowRight':
        e.preventDefault();
        setPlayerPosition(prev => ({ 
          ...prev, 
          x: Math.min(gameArea.width - 40, prev.x + moveSpeed) 
        }));
        break;
      case 'Enter':
        e.preventDefault();
        checkNPCInteraction();
        break;
    }
  }, [gameState, playerPosition]);

  // NPC와의 상호작용 확인
  const checkNPCInteraction = () => {
    const interactionDistance = 60;
    
    for (const npc of npcs) {
      const distance = Math.sqrt(
        Math.pow(playerPosition.x - npc.x, 2) + 
        Math.pow(playerPosition.y - npc.y, 2)
      );
      
      if (distance < interactionDistance && !invitedNPCs.has(npc.id)) {
        inviteNPC(npc);
        return;
      }
    }
  };

  // NPC 초대하기
  const inviteNPC = (npc) => {
    setInvitedNPCs(prev => new Set([...prev, npc.id]));
    showTemporaryMessage(`${npc.name}에게 초대장을 전달했습니다! 🎉`);
    
    // 사운드 효과 (간단한 비프음)
    try {
      const audioContext = new (window.AudioContext || window.webkitAudioContext)();
      const oscillator = audioContext.createOscillator();
      const gainNode = audioContext.createGain();
      
      oscillator.connect(gainNode);
      gainNode.connect(audioContext.destination);
      
      oscillator.frequency.setValueAtTime(800, audioContext.currentTime);
      gainNode.gain.setValueAtTime(0.1, audioContext.currentTime);
      gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.5);
      
      oscillator.start();
      oscillator.stop(audioContext.currentTime + 0.5);
    } catch (error) {
      console.log('Audio not supported');
    }
  };

  // 임시 메시지 표시
  const showTemporaryMessage = (message) => {
    setShowMessage(message);
    if (messageTimeout) clearTimeout(messageTimeout);
    const timeout = setTimeout(() => setShowMessage(''), 2000);
    setMessageTimeout(timeout);
  };

  // 키보드 이벤트 리스너 등록
  useEffect(() => {
    window.addEventListener('keydown', handleKeyPress);
    return () => window.removeEventListener('keydown', handleKeyPress);
  }, [handleKeyPress]);

  // 모든 NPC 초대 완료 시 파티 화면으로 전환
  useEffect(() => {
    if (invitedNPCs.size === npcs.length && gameState === 'playing') {
      setTimeout(() => {
        setGameState('party');
        showTemporaryMessage('모든 분들께 초대장을 전달했습니다! 🎊');
      }, 1000);
    }
  }, [invitedNPCs, gameState]);

  // 게임 재시작
  const resetGame = () => {
    setGameState('playing');
    setPlayerPosition({ x: 50, y: 50 });
    setInvitedNPCs(new Set());
    setResolution('');
    setShowMessage('');
  };

  // 다짐 제출
  const handleResolutionSubmit = () => {
    if (resolution.trim()) {
      setGameState('finished');
    }
  };

  return (
    <div style={{ 
      display: 'flex', 
      flexDirection: 'column', 
      alignItems: 'center', 
      padding: '20px',
      backgroundColor: '#f0f8ff',
      minHeight: '100vh',
      fontFamily: 'Arial, sans-serif'
    }}>
      <h1 style={{ 
        color: '#2c3e50', 
        marginBottom: '20px',
        textAlign: 'center'
      }}>
        🎉 입사 축하 파티 초대 게임 🎉
      </h1>

      {gameState === 'playing' && (
        <>
          <div style={{
            marginBottom: '10px',
            padding: '10px',
            backgroundColor: '#e8f4fd',
            borderRadius: '8px',
            textAlign: 'center'
          }}>
            <p style={{ margin: 0, fontSize: '14px' }}>
              방향키로 이동하고, 엔터키로 초대장을 전달하세요! 
              ({invitedNPCs.size}/{npcs.length}명 초대 완료)
            </p>
          </div>

          <div style={{
            position: 'relative',
            width: '600px',
            height: '400px',
            backgroundColor: '#90EE90',
            border: '3px solid #228B22',
            borderRadius: '10px',
            overflow: 'hidden',
            backgroundImage: 'linear-gradient(45deg, #90EE90 25%, transparent 25%), linear-gradient(-45deg, #90EE90 25%, transparent 25%), linear-gradient(45deg, transparent 75%, #90EE90 75%), linear-gradient(-45deg, transparent 75%, #90EE90 75%)',
            backgroundSize: '20px 20px',
            backgroundPosition: '0 0, 0 10px, 10px -10px, -10px 0px'
          }}>
            {/* 플레이어 */}
            <div style={{
              position: 'absolute',
              left: `${playerPosition.x}px`,
              top: `${playerPosition.y}px`,
              width: '40px',
              height: '40px',
              backgroundColor: '#4CAF50',
              border: '2px solid #2E7D32',
              borderRadius: '50%',
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'center',
              fontSize: '20px',
              zIndex: 10,
              transition: 'all 0.1s ease'
            }}>
              😊
            </div>

            {/* NPCs */}
            {npcs.map(npc => (
              <div key={npc.id} style={{
                position: 'absolute',
                left: `${npc.x}px`,
                top: `${npc.y}px`,
                width: '50px',
                height: '50px',
                backgroundColor: invitedNPCs.has(npc.id) ? '#FFD700' : '#FF6B6B',
                border: `3px solid ${invitedNPCs.has(npc.id) ? '#FFA500' : '#FF1744'}`,
                borderRadius: '50%',
                display: 'flex',
                alignItems: 'center',
                justifyContent: 'center',
                fontSize: '24px',
                transition: 'all 0.3s ease',
                transform: invitedNPCs.has(npc.id) ? 'scale(1.1)' : 'scale(1)',
                boxShadow: invitedNPCs.has(npc.id) ? '0 0 15px #FFD700' : 'none'
              }}>
                <div style={{ textAlign: 'center' }}>
                  <div>{npc.emoji}</div>
                  {invitedNPCs.has(npc.id) && (
                    <div style={{ 
                      fontSize: '12px', 
                      color: '#fff', 
                      fontWeight: 'bold',
                      marginTop: '2px'
                    }}>✓</div>
                  )}
                </div>
              </div>
            ))}

            {/* NPC 이름 표시 */}
            {npcs.map(npc => (
              <div key={`label-${npc.id}`} style={{
                position: 'absolute',
                left: `${npc.x - 10}px`,
                top: `${npc.y + 55}px`,
                fontSize: '12px',
                fontWeight: 'bold',
                color: '#2c3e50',
                textAlign: 'center',
                width: '70px',
                backgroundColor: 'rgba(255, 255, 255, 0.8)',
                padding: '2px',
                borderRadius: '4px'
              }}>
                {npc.name}
              </div>
            ))}
          </div>

          {showMessage && (
            <div style={{
              marginTop: '15px',
              padding: '15px',
              backgroundColor: '#4CAF50',
              color: 'white',
              borderRadius: '8px',
              fontSize: '16px',
              fontWeight: 'bold',
              animation: 'fadeIn 0.5s ease-in-out'
            }}>
              {showMessage}
            </div>
          )}
        </>
      )}

      {gameState === 'party' && (
        <div style={{
          textAlign: 'center',
          padding: '40px',
          backgroundColor: '#FFE4E1',
          borderRadius: '15px',
          border: '3px solid #FF69B4',
          maxWidth: '500px',
          animation: 'partyAnimation 2s ease-in-out infinite alternate'
        }}>
          <h2 style={{ color: '#FF1493', marginBottom: '20px' }}>
            🎊 축하합니다! 🎊
          </h2>
          <p style={{ fontSize: '18px', marginBottom: '30px', color: '#2c3e50' }}>
            모든 분들께 초대장을 전달했습니다!<br/>
            이제 나의 다짐을 입력해주세요.
          </p>
          
          <textarea
            value={resolution}
            onChange={(e) => setResolution(e.target.value)}
            placeholder="롯데캐피탈에서의 나의 다짐을 입력해주세요..."
            style={{
              width: '100%',
              height: '100px',
              padding: '15px',
              fontSize: '16px',
              borderRadius: '10px',
              border: '2px solid #FF69B4',
              resize: 'none',
              marginBottom: '20px',
              fontFamily: 'inherit'
            }}
          />
          
          <button
            onClick={handleResolutionSubmit}
            disabled={!resolution.trim()}
            style={{
              padding: '15px 30px',
              fontSize: '18px',
              fontWeight: 'bold',
              backgroundColor: resolution.trim() ? '#4CAF50' : '#cccccc',
              color: 'white',
              border: 'none',
              borderRadius: '25px',
              cursor: resolution.trim() ? 'pointer' : 'not-allowed',
              transition: 'all 0.3s ease',
              transform: resolution.trim() ? 'scale(1)' : 'scale(0.95)'
            }}
          >
            다짐 완료! 🚀
          </button>
        </div>
      )}

      {gameState === 'finished' && (
        <div style={{
          textAlign: 'center',
          padding: '40px',
          backgroundColor: '#E8F5E8',
          borderRadius: '15px',
          border: '3px solid #4CAF50',
          maxWidth: '500px'
        }}>
          <h2 style={{ color: '#2E7D32', marginBottom: '20px' }}>
            🎉 환영합니다! 🎉
          </h2>
          
          <div style={{
            backgroundColor: '#F1F8E9',
            padding: '20px',
            borderRadius: '10px',
            marginBottom: '20px',
            border: '2px solid #8BC34A'
          }}>
            <p style={{ fontSize: '16px', color: '#2c3e50', marginBottom: '10px' }}>
              <strong>나의 다짐:</strong>
            </p>
            <p style={{ 
              fontSize: '18px', 
              color: '#2E7D32', 
              fontStyle: 'italic',
              lineHeight: '1.5'
            }}>
              "{resolution}"
            </p>
          </div>
          
          <h3 style={{ 
            color: '#FF6B35', 
            fontSize: '24px', 
            fontWeight: 'bold',
            marginBottom: '30px',
            textShadow: '2px 2px 4px rgba(0,0,0,0.1)'
          }}>
            🏢 당신은 롯데캐피탈의 일원이 되었습니다! 🏢
          </h3>
          
          <button
            onClick={resetGame}
            style={{
              padding: '15px 30px',
              fontSize: '18px',
              fontWeight: 'bold',
              backgroundColor: '#2196F3',
              color: 'white',
              border: 'none',
              borderRadius: '25px',
              cursor: 'pointer',
              transition: 'all 0.3s ease',
              marginRight: '10px'
            }}
          >
            🔄 다시 시작하기
          </button>
        </div>
      )}

      <style>{`
        @keyframes fadeIn {
          from { opacity: 0; transform: translateY(10px); }
          to { opacity: 1; transform: translateY(0); }
        }

        @keyframes partyAnimation {
          from { transform: scale(1); }
          to { transform: scale(1.02); }
        }

        button:hover {
          transform: scale(1.05) !important;
          box-shadow: 0 4px 15px rgba(0,0,0,0.2);
        }
      `}</style>
    </div>
  );
};

export default InvitationGame;
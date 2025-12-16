@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --background: 0 0% 0%;
  --foreground: 0 0% 100%;
  --card: 0 0% 0%;
  --card-foreground: 0 0% 100%;
  --popover: 0 0% 0%;
  --popover-foreground: 0 0% 100%;
  --primary: 72 100% 63%;
  --primary-foreground: 0 0% 0%;
  --secondary: 0 0% 15%;
  --secondary-foreground: 0 0% 100%;
  --muted: 0 0% 15%;
  --muted-foreground: 0 0% 64%;
  --accent: 72 100% 63%;
  --accent-foreground: 0 0% 0%;
  --destructive: 0 84% 60%;
  --destructive-foreground: 0 0% 100%;
  --border: 0 0% 20%;
  --input: 0 0% 20%;
  --ring: 72 100% 63%;
  --radius: 0.5rem;
}

* {
  @apply border-border;
}

body {
  @apply bg-background text-foreground;
  font-feature-settings: "rlig" 1, "calt" 1;
}

/* Custom animations */
@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fade-in {
  animation: fade-in 0.6s ease-out;
}

/* Smooth transitions */
* {
  transition-property: color, background-color, border-color, text-decoration-color, fill, stroke;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 150ms;
}

/* Custom scrollbar */
::-webkit-scrollbar {
  width: 10px;
}

::-webkit-scrollbar-track {
  background: #000000;
}

::-webkit-scrollbar-thumb {
  background: #333333;
  border-radius: 5px;
}

::-webkit-scrollbar-thumb:hover {
  background: #e2ff57;
}

/* Button hover effects */
button {
  position: relative;
  overflow: hidden;
}

button::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  border-radius: 50%;
  background: rgba(226, 255, 87, 0.3);
  transform: translate(-50%, -50%);
  transition: width 0.6s, height 0.6s;
}

button:hover::before {
  width: 300px;
  height: 300px;
}

/* Card hover lift effect */
.hover\:scale-102:hover {
  transform: scale(1.02);
}

/* Progress bar custom styling */
[role="progressbar"] {
  background: linear-gradient(90deg, #e2ff57 0%, #a8c700 100%);
}

/* Mobile optimizations */
@media (max-width: 768px) {
  body {
    font-size: 16px;
  }
  
  h1 {
    font-size: 2rem;
  }
  
  h2 {
    font-size: 1.5rem;
  }
}

/* Ensure proper mobile spacing */
.container {
  @apply px-4 md:px-6 lg:px-8;
}

/* Prevent text selection on UI elements */
button, .trophy-display {
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
import type { Metadata } from "next";
import localFont from "next/font/local";
import "./globals.css";
import { ResponseLogger } from "@/components/response-logger";
import { cookies } from "next/headers";
import { ReadyNotifier } from "@/components/ready-notifier";
import FarcasterWrapper from "@/components/FarcasterWrapper";

const geistSans = localFont({
  src: "./fonts/GeistVF.woff",
  variable: "--font-geist-sans",
  weight: "100 900",
});
const geistMono = localFont({
  src: "./fonts/GeistMonoVF.woff",
  variable: "--font-geist-mono",
  weight: "100 900",
});

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  const requestId = cookies().get("x-request-id")?.value;

  return (
        <html lang="en">
          <head>
            {requestId && <meta name="x-request-id" content={requestId} />}
          </head>
          <body
            className={`${geistSans.variable} ${geistMono.variable} antialiased`}
          >
            {/* Do not remove this component, we use it to notify the parent that the mini-app is ready */}
            <ReadyNotifier />
            
      <FarcasterWrapper>
        {children}
      </FarcasterWrapper>
      
            <ResponseLogger />
          </body>
        </html>
      );
}

export const metadata: Metadata = {
        title: "OFC Quiz Championship",
        description: "Test your OneFootball knowledge in this interactive quiz! Win trophies, share results, and enjoy a football-themed experience with real-time scoring and dynamic challenges.",
        other: { "fc:frame": JSON.stringify({"version":"next","imageUrl":"https://usdozf7pplhxfvrl.public.blob.vercel-storage.com/thumbnail_e4bff3c9-a02c-41e7-aea5-7f6e1a823ea0-FDEiakQfcRN74qt3F5lFUcCEzpYUuD","button":{"title":"Open with Ohara","action":{"type":"launch_frame","name":"OFC Quiz Championship","url":"https://farmer-station-266.app.ohara.ai","splashImageUrl":"https://usdozf7pplhxfvrl.public.blob.vercel-storage.com/farcaster/splash_images/splash_image1.svg","splashBackgroundColor":"#ffffff"}}}
        ) }
    };
'use client'
import React, { useState, useEffect } from 'react';
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';
import { Progress } from '@/components/ui/progress';
import { Menu, X, Home, Info } from 'lucide-react';
import QuizQuestion from '@/components/QuizQuestion';
import QuizResult from '@/components/QuizResult';
import { quizData } from '@/data/quizData';
import type { Question } from '@/types/quiz';
import { sdk } from "@farcaster/miniapp-sdk";
import { useAddMiniApp } from "@/hooks/useAddMiniApp";
import { useQuickAuth } from "@/hooks/useQuickAuth";
import { useIsInFarcaster } from "@/hooks/useIsInFarcaster";

export default function Page(): JSX.Element {
    const { addMiniApp } = useAddMiniApp();
    const isInFarcaster = useIsInFarcaster()
    useQuickAuth(isInFarcaster)
    useEffect(() => {
      const tryAddMiniApp = async () => {
        try {
          await addMiniApp()
        } catch (error) {
          console.error('Failed to add mini app:', error)
        }

      }

    

      tryAddMiniApp()
    }, [addMiniApp])
    useEffect(() => {
      const initializeFarcaster = async () => {
        try {
          await new Promise(resolve => setTimeout(resolve, 100))
          
          if (document.readyState !== 'complete') {
            await new Promise<void>(resolve => {
              if (document.readyState === 'complete') {
                resolve()
              } else {
                window.addEventListener('load', () => resolve(), { once: true })
              }

            })
          }

    

          await sdk.actions.ready()
          console.log('Farcaster SDK initialized successfully - app fully loaded')
        } catch (error) {
          console.error('Failed to initialize Farcaster SDK:', error)
          
          setTimeout(async () => {
            try {
              await sdk.actions.ready()
              console.log('Farcaster SDK initialized on retry')
            } catch (retryError) {
              console.error('Farcaster SDK retry failed:', retryError)
            }

          }, 1000)
        }

      }

    

      initializeFarcaster()
    }, [])
  const [quizStarted, setQuizStarted] = useState<boolean>(false);
  const [currentQuestion, setCurrentQuestion] = useState<number>(0);
  const [score, setScore] = useState<number>(0);
  const [selectedAnswer, setSelectedAnswer] = useState<number | null>(null);
  const [showResult, setShowResult] = useState<boolean>(false);
  const [questions, setQuestions] = useState<Question[]>([]);
  const [mobileMenuOpen, setMobileMenuOpen] = useState<boolean>(false);
  const [isAnswered, setIsAnswered] = useState<boolean>(false);

  useEffect(() => {
    // Shuffle and select 10 unique questions on mount
    const shuffled = [...quizData].sort(() => Math.random() - 0.5).slice(0, 10);
    setQuestions(shuffled);
  }, []);

  const handleStartQuiz = (): void => {
    setQuizStarted(true);
    setCurrentQuestion(0);
    setScore(0);
    setSelectedAnswer(null);
    setShowResult(false);
    setIsAnswered(false);
    
    // Reshuffle questions for new quiz
    const shuffled = [...quizData].sort(() => Math.random() - 0.5).slice(0, 10);
    setQuestions(shuffled);
  };

  const handleAnswerSelect = (answerIndex: number): void => {
    if (isAnswered) return;
    
    setSelectedAnswer(answerIndex);
    setIsAnswered(true);

    if (answerIndex === questions[currentQuestion].correctAnswer) {
      setScore(score + 1);
    }

    setTimeout(() => {
      if (currentQuestion < questions.length - 1) {
        setCurrentQuestion(currentQuestion + 1);
        setSelectedAnswer(null);
        setIsAnswered(false);
      } else {
        setShowResult(true);
      }
    }, 1500);
  };

  const handleRetakeQuiz = (): void => {
    handleStartQuiz();
  };

  const handleBackToHome = (): void => {
    setQuizStarted(false);
    setShowResult(false);
    setCurrentQuestion(0);
    setScore(0);
    setSelectedAnswer(null);
    setIsAnswered(false);
  };

  const progressPercentage = ((currentQuestion + 1) / questions.length) * 100;

  if (showResult) {
    return <QuizResult score={score} totalQuestions={questions.length} onRetake={handleRetakeQuiz} onBackToHome={handleBackToHome} />;
  }

  return (
    <div className="min-h-screen bg-black text-white relative overflow-hidden">
      {/* Stadium Background */}
      <div className="fixed inset-0 z-0 opacity-20">
        <div className="absolute inset-0 bg-gradient-to-b from-green-900/30 via-black to-black" />
        <div className="absolute bottom-0 left-0 right-0 h-1/2 bg-gradient-to-t from-green-950/20 to-transparent" />
        {/* Stadium lights effect */}
        <div className="absolute top-0 left-1/4 w-32 h-32 bg-yellow-400/10 rounded-full blur-3xl" />
        <div className="absolute top-0 right-1/4 w-32 h-32 bg-yellow-400/10 rounded-full blur-3xl" />
      </div>

      {/* Header */}
      <header className="relative z-10 border-b border-[#e2ff57]/20 bg-black/80 backdrop-blur-sm">
        <div className="container mx-auto px-4 py-4 flex justify-between items-center">
          <div className="flex items-center gap-4">
            <img src="https://i.ibb.co/rGfwfvx3/TMn0-Dvx-400x400.png" alt="OFC Logo" className="h-12 w-12 object-contain" />
            <h1 className="text-2xl font-bold">
              <span className="text-[#e2ff57]">OFC</span> Quiz
            </h1>
          </div>

          {/* Desktop Navigation */}
          <nav className="hidden md:flex items-center gap-6">
            <button onClick={handleBackToHome} className="flex items-center gap-2 hover:text-[#e2ff57] transition-colors">
              <Home size={20} />
              <span>Home</span>
            </button>
            <button className="flex items-center gap-2 hover:text-[#e2ff57] transition-colors">
              <Info size={20} />
              <span>About OFC</span>
            </button>
            <img src="https://i.ibb.co/JRJBLNk6/Ke-RAoa-XO-400x400.jpg" alt="OneFootball Logo" className="h-8 w-8 object-contain ml-4" />
          </nav>

          {/* Mobile Menu Button */}
          <button 
            onClick={() => setMobileMenuOpen(!mobileMenuOpen)}
            className="md:hidden text-[#e2ff57] hover:text-white transition-colors"
          >
            {mobileMenuOpen ? <X size={28} /> : <Menu size={28} />}
          </button>
        </div>

        {/* Mobile Navigation */}
        {mobileMenuOpen && (
          <nav className="md:hidden bg-black/95 border-t border-[#e2ff57]/20 py-4">
            <div className="container mx-auto px-4 flex flex-col gap-4">
              <button onClick={() => { handleBackToHome(); setMobileMenuOpen(false); }} className="flex items-center gap-2 hover:text-[#e2ff57] transition-colors py-2">
                <Home size={20} />
                <span>Home</span>
              </button>
              <button onClick={() => setMobileMenuOpen(false)} className="flex items-center gap-2 hover:text-[#e2ff57] transition-colors py-2">
                <Info size={20} />
                <span>About OFC</span>
              </button>
              <div className="flex items-center gap-2 py-2">
                <img src="https://i.ibb.co/JRJBLNk6/Ke-RAoa-XO-400x400.jpg" alt="OneFootball Logo" className="h-6 w-6 object-contain" />
                <span className="text-sm text-gray-400">Powered by OneFootball</span>
              </div>
            </div>
          </nav>
        )}
      </header>

      {/* Main Content */}
      <main className="relative z-10 container mx-auto px-4 py-8 md:py-12">
        {!quizStarted ? (
          // Hero Section
          <div className="max-w-4xl mx-auto text-center pt-12 md:pt-20 pb-20">
            <div className="mb-8 animate-fade-in">
              <img 
                src="https://i.ibb.co/rGfwfvx3/TMn0-Dvx-400x400.png" 
                alt="OFC Logo" 
                className="h-32 w-32 md:h-40 md:w-40 mx-auto mb-6 drop-shadow-2xl"
              />
            </div>

            <h2 className="text-4xl md:text-6xl font-bold mb-6 leading-tight">
              Test Your <span className="text-[#e2ff57]">OneFootball Club</span> Knowledge
            </h2>

            <p className="text-lg md:text-xl text-gray-300 mb-4 max-w-2xl mx-auto leading-relaxed">
              Think you know everything about <span className="text-[#e2ff57] font-bold">OneFootball Club</span>? 
            </p>

            <p className="text-base md:text-lg text-gray-400 mb-8 max-w-2xl mx-auto">
              Take this <span className="font-bold">10-question challenge</span> and prove your fandom! 
              Earn your trophy and share your results with the community.
            </p>

            {/* Features Grid */}
            <div className="grid grid-cols-1 md:grid-cols-3 gap-4 max-w-3xl mx-auto mb-12">
              <Card className="bg-gradient-to-br from-green-900/30 to-black border-[#e2ff57]/30 p-6 text-center hover:border-[#e2ff57]/60 transition-all">
                <div className="text-4xl mb-2">⚽</div>
                <h3 className="text-[#e2ff57] font-bold mb-2">10 Questions</h3>
                <p className="text-sm text-gray-400">Progressive difficulty</p>
              </Card>

              <Card className="bg-gradient-to-br from-yellow-900/30 to-black border-[#e2ff57]/30 p-6 text-center hover:border-[#e2ff57]/60 transition-all">
                <div className="text-4xl mb-2">🏆</div>
                <h3 className="text-[#e2ff57] font-bold mb-2">Earn Trophies</h3>
                <p className="text-sm text-gray-400">Gold, Silver, or Bronze</p>
              </Card>

              <Card className="bg-gradient-to-br from-blue-900/30 to-black border-[#e2ff57]/30 p-6 text-center hover:border-[#e2ff57]/60 transition-all">
                <div className="text-4xl mb-2">📤</div>
                <h3 className="text-[#e2ff57] font-bold mb-2">Share Results</h3>
                <p className="text-sm text-gray-400">Download & post</p>
              </Card>
            </div>

            <div className="flex flex-col sm:flex-row items-center justify-center gap-4">
              <Button 
                onClick={handleStartQuiz}
                size="lg"
                className="bg-[#e2ff57] text-black hover:bg-[#e2ff57]/90 text-xl px-12 py-6 rounded-full font-bold shadow-xl hover:scale-105 transition-transform"
              >
                Start Quiz ⚡
              </Button>
              
              <Button 
                onClick={() => window.open('https://x.com/hassaniazii', '_blank')}
                size="lg"
                className="bg-blue-500 text-white hover:bg-blue-600 text-xl px-12 py-6 rounded-full font-bold shadow-xl hover:scale-105 transition-transform flex items-center gap-2"
              >
                <svg className="w-6 h-6" fill="currentColor" viewBox="0 0 24 24">
                  <path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"/>
                </svg>
                Follow @hassaniazii
              </Button>
            </div>

            {/* About Section - Enhanced */}
            <div className="mt-24 max-w-5xl mx-auto">
              <div className="text-center mb-12">
                <h3 className="text-3xl md:text-5xl font-bold mb-6">
                  Welcome to <span className="text-[#e2ff57]">OneFootball Club</span>
                </h3>
                <p className="text-xl md:text-2xl text-gray-300 font-semibold mb-4">
                  The <span className="text-[#e2ff57]">World's Largest Digital Football Community</span>
                </p>
                <p className="text-base md:text-lg text-gray-400 max-w-3xl mx-auto leading-relaxed">
                  We're <span className="text-white font-bold">revolutionizing football fandom</span> by putting fans at the center, 
                  redistributing value, and creating the <span className="text-[#e2ff57] font-bold">most engaged sports community</span> in the world.
                </p>
              </div>

              {/* Stats Grid */}
              <div className="grid grid-cols-2 md:grid-cols-4 gap-4 mb-12">
                <Card className="bg-gradient-to-br from-[#e2ff57]/20 to-black border-[#e2ff57]/40 p-6 text-center hover:scale-105 transition-transform">
                  <div className="text-3xl md:text-4xl font-bold text-[#e2ff57] mb-2">500M+</div>
                  <div className="text-xs md:text-sm text-gray-400">Global Football Fans</div>
                </Card>
                <Card className="bg-gradient-to-br from-green-500/20 to-black border-[#e2ff57]/40 p-6 text-center hover:scale-105 transition-transform">
                  <div className="text-3xl md:text-4xl font-bold text-[#e2ff57] mb-2">Web3</div>
                  <div className="text-xs md:text-sm text-gray-400">Blockchain Powered</div>
                </Card>
                <Card className="bg-gradient-to-br from-blue-500/20 to-black border-[#e2ff57]/40 p-6 text-center hover:scale-105 transition-transform">
                  <div className="text-3xl md:text-4xl font-bold text-[#e2ff57] mb-2">$OFC</div>
                  <div className="text-xs md:text-sm text-gray-400">Community Token</div>
                </Card>
                <Card className="bg-gradient-to-br from-purple-500/20 to-black border-[#e2ff57]/40 p-6 text-center hover:scale-105 transition-transform">
                  <div className="text-3xl md:text-4xl font-bold text-[#e2ff57] mb-2">∞</div>
                  <div className="text-xs md:text-sm text-gray-400">Unlimited Potential</div>
                </Card>
              </div>

              {/* Bullish Features */}
              <div className="grid md:grid-cols-2 gap-6 mb-12">
                <Card className="bg-gradient-to-br from-black via-green-900/20 to-black border-[#e2ff57]/30 p-8 hover:border-[#e2ff57]/60 transition-all">
                  <div className="flex items-start gap-4">
                    <div className="text-4xl">🚀</div>
                    <div>
                      <h4 className="text-xl font-bold mb-3 text-[#e2ff57]">Fan-First Revolution</h4>
                      <p className="text-gray-300 leading-relaxed">
                        We're <span className="font-bold text-white">disrupting traditional football economics</span> by 
                        redistributing value directly to fans. Your engagement, your data, <span className="text-[#e2ff57] font-bold">your rewards</span>.
                      </p>
                    </div>
                  </div>
                </Card>

                <Card className="bg-gradient-to-br from-black via-blue-900/20 to-black border-[#e2ff57]/30 p-8 hover:border-[#e2ff57]/60 transition-all">
                  <div className="flex items-start gap-4">
                    <div className="text-4xl">🎮</div>
                    <div>
                      <h4 className="text-xl font-bold mb-3 text-[#e2ff57]">Gamification 2.0</h4>
                      <p className="text-gray-300 leading-relaxed">
                        <span className="font-bold text-white">Earn while you engage</span>. Every prediction, every interaction, 
                        every moment counts toward <span className="text-[#e2ff57] font-bold">real rewards</span> and exclusive perks.
                      </p>
                    </div>
                  </div>
                </Card>

                <Card className="bg-gradient-to-br from-black via-purple-900/20 to-black border-[#e2ff57]/30 p-8 hover:border-[#e2ff57]/60 transition-all">
                  <div className="flex items-start gap-4">
                    <div className="text-4xl">⚡</div>
                    <div>
                      <h4 className="text-xl font-bold mb-3 text-[#e2ff57]">True Digital Ownership</h4>
                      <p className="text-gray-300 leading-relaxed">
                        Your <span className="font-bold text-white">.football ID</span> is your <span className="text-[#e2ff57] font-bold">portable fan passport</span>. 
                        Own your identity, collect OneFootball Heads, and build your <span className="font-bold">legacy on-chain</span>.
                      </p>
                    </div>
                  </div>
                </Card>

                <Card className="bg-gradient-to-br from-black via-yellow-900/20 to-black border-[#e2ff57]/30 p-8 hover:border-[#e2ff57]/60 transition-all">
                  <div className="flex items-start gap-4">
                    <div className="text-4xl">🌍</div>
                    <div>
                      <h4 className="text-xl font-bold mb-3 text-[#e2ff57]">Global Community Power</h4>
                      <p className="text-gray-300 leading-relaxed">
                        Join <span className="font-bold text-white">the world's most passionate football community</span>. 
                        Compete on leaderboards, complete challenges, and <span className="text-[#e2ff57] font-bold">shape the future</span> of fan engagement.
                      </p>
                    </div>
                  </div>
                </Card>
              </div>

              {/* Bold Statement */}
              <div className="text-center bg-gradient-to-r from-transparent via-[#e2ff57]/10 to-transparent border-y border-[#e2ff57]/30 py-8 px-6">
                <p className="text-xl md:text-2xl font-bold text-white mb-2">
                  "Football belongs to the fans. <span className="text-[#e2ff57]">We're making that a reality.</span>"
                </p>
                <p className="text-sm text-gray-400">
                  OneFootball Club is building the infrastructure for <span className="text-[#e2ff57]">the future of fan engagement</span>
                </p>
              </div>
            </div>
          </div>
        ) : (
          // Quiz Section
          <div className="max-w-3xl mx-auto">
            {/* Progress Bar */}
            <div className="mb-8">
              <div className="flex justify-between items-center mb-3">
                <span className="text-sm text-gray-400">
                  Question {currentQuestion + 1} of {questions.length}
                </span>
                <span className="text-sm text-[#e2ff57] font-bold">
                  ⚽ Score: {score}
                </span>
              </div>
              <div className="relative">
                <Progress value={progressPercentage} className="h-3 bg-gray-800" />
                {/* Football icon at progress position */}
                <div 
                  className="absolute top-1/2 -translate-y-1/2 text-xl transition-all duration-300"
                  style={{ left: `calc(${progressPercentage}% - 12px)` }}
                >
                  ⚽
                </div>
              </div>
            </div>

            {/* Question Card */}
            {questions.length > 0 && (
              <QuizQuestion
                key={currentQuestion}
                question={questions[currentQuestion]}
                questionNumber={currentQuestion + 1}
                selectedAnswer={selectedAnswer}
                isAnswered={isAnswered}
                onAnswerSelect={handleAnswerSelect}
              />
            )}
          </div>
        )}
      </main>

      {/* Footer */}
      <footer className="relative z-10 border-t border-[#e2ff57]/20 bg-black/80 backdrop-blur-sm py-6 mt-20">
        <div className="container mx-auto px-4 text-center">
          <div className="flex items-center justify-center gap-3 mb-3">
            <img src="https://i.ibb.co/JRJBLNk6/Ke-RAoa-XO-400x400.jpg" alt="OneFootball Logo" className="h-6 w-6 object-contain" />
            <span className="text-sm text-gray-400">Powered by OneFootball</span>
          </div>
          <p className="text-xs text-gray-500">
            © 2024 OneFootball Club. Built for the world's largest football community.
          </p>
        </div>
      </footer>
    </div>
  );
}
'use client';

import React from 'react';
import { Card } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { CheckCircle2, XCircle } from 'lucide-react';
import type { Question } from '@/types/quiz';

interface QuizQuestionProps {
  question: Question;
  questionNumber: number;
  selectedAnswer: number | null;
  isAnswered: boolean;
  onAnswerSelect: (answerIndex: number) => void;
}

export default function QuizQuestion({
  question,
  questionNumber,
  selectedAnswer,
  isAnswered,
  onAnswerSelect
}: QuizQuestionProps): JSX.Element {
  const getDifficultyColor = (difficulty: string): string => {
    switch (difficulty) {
      case 'easy':
        return 'text-green-400';
      case 'medium':
        return 'text-yellow-400';
      case 'hard':
        return 'text-red-400';
      default:
        return 'text-gray-400';
    }
  };

  const getDifficultyLabel = (difficulty: string): string => {
    return difficulty.charAt(0).toUpperCase() + difficulty.slice(1);
  };

  return (
    <Card className="bg-gradient-to-br from-green-900/20 via-black to-black border-2 border-[#e2ff57]/40 p-6 md:p-8 shadow-2xl relative overflow-hidden">
      {/* Goal Post Frame Effect */}
      <div className="absolute top-0 left-0 w-full h-2 bg-gradient-to-r from-transparent via-[#e2ff57]/30 to-transparent" />
      <div className="absolute bottom-0 left-0 w-full h-2 bg-gradient-to-r from-transparent via-[#e2ff57]/30 to-transparent" />
      <div className="absolute top-0 left-0 w-2 h-full bg-gradient-to-b from-transparent via-[#e2ff57]/30 to-transparent" />
      <div className="absolute top-0 right-0 w-2 h-full bg-gradient-to-b from-transparent via-[#e2ff57]/30 to-transparent" />

      {/* Question Number */}
      <div className="mb-6">
        <span className="text-xs uppercase tracking-wider text-gray-500 font-bold">
          Question {questionNumber}
        </span>
      </div>

      {/* Question */}
      <h3 className="text-xl md:text-2xl font-bold mb-8 leading-relaxed">
        {question.question}
      </h3>

      {/* Answer Options */}
      <div className="space-y-4">
        {question.options.map((option: string, index: number) => {
          const isCorrect = index === question.correctAnswer;
          const isSelected = selectedAnswer === index;
          const showFeedback = isAnswered && isSelected;

          let buttonClasses = 'w-full text-left p-4 md:p-5 rounded-lg border-2 transition-all duration-300 font-medium text-base md:text-lg';
          
          if (!isAnswered) {
            buttonClasses += ' bg-gray-900/50 border-gray-700 hover:border-[#e2ff57] hover:bg-gray-800/70 hover:scale-102';
          } else if (isSelected) {
            if (isCorrect) {
              buttonClasses += ' bg-green-900/40 border-green-500 scale-102';
            } else {
              buttonClasses += ' bg-red-900/40 border-red-500';
            }
          } else if (isAnswered && isCorrect) {
            buttonClasses += ' bg-green-900/20 border-green-500/50';
          } else {
            buttonClasses += ' bg-gray-900/30 border-gray-700/50 opacity-50';
          }

          return (
            <button
              key={index}
              onClick={() => onAnswerSelect(index)}
              disabled={isAnswered}
              className={buttonClasses}
            >
              <div className="flex items-center justify-between">
                <span className="flex-1 pr-4">
                  <span className="text-[#e2ff57] font-bold mr-3">
                    {String.fromCharCode(65 + index)}.
                  </span>
                  {option}
                </span>
                
                {showFeedback && (
                  <span className="flex-shrink-0">
                    {isCorrect ? (
                      <CheckCircle2 className="text-green-400" size={24} />
                    ) : (
                      <XCircle className="text-red-400" size={24} />
                    )}
                  </span>
                )}

                {isAnswered && isCorrect && !isSelected && (
                  <CheckCircle2 className="text-green-400 opacity-50" size={24} />
                )}
              </div>
            </button>
          );
        })}
      </div>

      {/* Feedback Message */}
      {isAnswered && (
        <div className={`mt-6 p-4 rounded-lg text-center font-medium ${
          selectedAnswer === question.correctAnswer
            ? 'bg-green-900/30 text-green-300 border border-green-500/30'
            : 'bg-red-900/30 text-red-300 border border-red-500/30'
        }`}>
          {selectedAnswer === question.correctAnswer ? (
            <>⚽ Correct! Great knowledge!</>
          ) : (
            <>💭 Not quite! Keep learning!</>
          )}
        </div>
      )}

      {/* Blockchain hexagon pattern decoration */}
      <div className="absolute bottom-4 right-4 opacity-5">
        <svg width="80" height="80" viewBox="0 0 80 80" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M40 10L60 22.5V47.5L40 60L20 47.5V22.5L40 10Z" stroke="#e2ff57" strokeWidth="2"/>
          <path d="M40 20L52 27V41L40 48L28 41V27L40 20Z" stroke="#e2ff57" strokeWidth="2"/>
        </svg>
      </div>
    </Card>
  );
}
'use client';

import React, { useEffect, useState, useRef } from 'react';
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Home, RefreshCw, Download, Upload } from 'lucide-react';
import TrophyDisplay from '@/components/TrophyDisplay';
import type { TrophyData } from '@/types/quiz';

interface QuizResultProps {
  score: number;
  totalQuestions: number;
  onRetake: () => void;
  onBackToHome: () => void;
}

export default function QuizResult({
  score,
  totalQuestions,
  onRetake,
  onBackToHome
}: QuizResultProps): JSX.Element {
  const [trophy, setTrophy] = useState<TrophyData | null>(null);
  const [showConfetti, setShowConfetti] = useState<boolean>(false);
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const [isGenerating, setIsGenerating] = useState<boolean>(false);
  
  // Personalization state
  const [showPersonalizationForm, setShowPersonalizationForm] = useState<boolean>(true);
  const [userName, setUserName] = useState<string>('');
  const [userImage, setUserImage] = useState<string | null>(null);
  const fileInputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    const trophyData = getTrophyData(score, totalQuestions);
    setTrophy(trophyData);
    
    if (trophyData.type !== 'tryagain') {
      setShowConfetti(true);
      setTimeout(() => setShowConfetti(false), 5000);
    }
  }, [score, totalQuestions]);

  const getTrophyData = (score: number, total: number): TrophyData => {
    if (score >= 9) {
      return {
        type: 'gold',
        title: 'Gold Trophy! 🥇',
        message: 'Outstanding! You\'re a true OFC expert!',
        emoji: '🏆',
        color: '#FFD700'
      };
    } else if (score >= 7) {
      return {
        type: 'silver',
        title: 'Silver Trophy! 🥈',
        message: 'Excellent work! You know your OFC!',
        emoji: '🥈',
        color: '#C0C0C0'
      };
    } else if (score >= 5) {
      return {
        type: 'bronze',
        title: 'Bronze Trophy! 🥉',
        message: 'Good effort! Keep learning!',
        emoji: '🥉',
        color: '#CD7F32'
      };
    } else {
      return {
        type: 'tryagain',
        title: 'Try Again! 📚',
        message: 'Keep learning about OFC and try again!',
        emoji: '💪',
        color: '#666666'
      };
    }
  };

  const handleImageUpload = (event: React.ChangeEvent<HTMLInputElement>): void => {
    const file = event.target.files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e: ProgressEvent<FileReader>) => {
        const result = e.target?.result;
        if (typeof result === 'string') {
          setUserImage(result);
        }
      };
      reader.readAsDataURL(file);
    }
  };

  const handleSubmitPersonalization = (): void => {
    if (userName.trim()) {
      setShowPersonalizationForm(false);
    }
  };

  const handleDownloadImage = async (): Promise<void> => {
    if (!canvasRef.current || !trophy) return;
    
    setIsGenerating(true);
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    if (!ctx) return;

    // Set canvas size
    canvas.width = 1200;
    canvas.height = 1200;

    // Background
    ctx.fillStyle = '#000000';
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Stadium gradient background
    const gradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
    gradient.addColorStop(0, 'rgba(22, 101, 52, 0.3)');
    gradient.addColorStop(0.5, 'rgba(0, 0, 0, 0.8)');
    gradient.addColorStop(1, 'rgba(20, 83, 45, 0.2)');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Spotlight effects
    const spotlightGradient = ctx.createRadialGradient(300, 200, 100, 300, 200, 400);
    spotlightGradient.addColorStop(0, 'rgba(226, 255, 87, 0.2)');
    spotlightGradient.addColorStop(1, 'transparent');
    ctx.fillStyle = spotlightGradient;
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    const spotlightGradient2 = ctx.createRadialGradient(900, 200, 100, 900, 200, 400);
    spotlightGradient2.addColorStop(0, 'rgba(226, 255, 87, 0.2)');
    spotlightGradient2.addColorStop(1, 'transparent');
    ctx.fillStyle = spotlightGradient2;
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Border
    ctx.strokeStyle = '#e2ff57';
    ctx.lineWidth = 8;
    ctx.strokeRect(40, 40, canvas.width - 80, canvas.height - 80);

    // Load and draw OFC logo
    const logo = new Image();
    logo.crossOrigin = 'anonymous';
    logo.src = 'https://i.ibb.co/rGfwfvx3/TMn0-Dvx-400x400.png';
    await new Promise((resolve) => {
      logo.onload = resolve;
    });
    ctx.drawImage(logo, (canvas.width - 120) / 2, 100, 120, 120);

    // User image (if provided)
    if (userImage) {
      try {
        const userImg = new Image();
        userImg.src = userImage;
        await new Promise((resolve) => {
          userImg.onload = resolve;
        });
        
        // Draw circular user image
        ctx.save();
        ctx.beginPath();
        ctx.arc(canvas.width / 2, 300, 80, 0, Math.PI * 2);
        ctx.closePath();
        ctx.clip();
        ctx.drawImage(userImg, canvas.width / 2 - 80, 220, 160, 160);
        ctx.restore();
        
        // Circle border around user image
        ctx.strokeStyle = '#e2ff57';
        ctx.lineWidth = 6;
        ctx.beginPath();
        ctx.arc(canvas.width / 2, 300, 80, 0, Math.PI * 2);
        ctx.stroke();
      } catch (error) {
        console.error('Error loading user image:', error);
      }
    }

    // User name (if provided)
    if (userName) {
      ctx.font = 'bold 48px Arial';
      ctx.fillStyle = '#ffffff';
      ctx.textAlign = 'center';
      ctx.fillText(userName, canvas.width / 2, userImage ? 420 : 300);
    }

    // Trophy emoji
    ctx.font = 'bold 140px Arial';
    ctx.textAlign = 'center';
    ctx.fillText(trophy.emoji, canvas.width / 2, userImage || userName ? 580 : 500);

    // Trophy title
    ctx.font = 'bold 70px Arial';
    ctx.fillStyle = trophy.color;
    ctx.fillText(trophy.title.replace(/[^\w\s]/gi, ''), canvas.width / 2, userImage || userName ? 700 : 630);

    // Score
    ctx.font = 'bold 100px Arial';
    ctx.fillStyle = '#e2ff57';
    ctx.fillText(`${score}/${totalQuestions}`, canvas.width / 2, userImage || userName ? 840 : 790);

    // Percentage
    const percentage = Math.round((score / totalQuestions) * 100);
    ctx.font = 'bold 50px Arial';
    ctx.fillStyle = '#ffffff';
    ctx.fillText(`${percentage}% Correct`, canvas.width / 2, userImage || userName ? 920 : 880);

    // Message
    ctx.font = '36px Arial';
    ctx.fillStyle = '#cccccc';
    ctx.fillText('I completed the OFC Quiz!', canvas.width / 2, userImage || userName ? 1000 : 980);

    // OneFootball branding
    ctx.font = 'bold 32px Arial';
    ctx.fillStyle = '#e2ff57';
    ctx.fillText('OneFootball Club', canvas.width / 2, userImage || userName ? 1100 : 1100);

    // Download
    canvas.toBlob((blob) => {
      if (blob) {
        const url = URL.createObjectURL(blob);
        const link = document.createElement('a');
        link.href = url;
        link.download = `ofc-quiz-result-${score}-${totalQuestions}.png`;
        link.click();
        URL.revokeObjectURL(url);
      }
      setIsGenerating(false);
    });
  };

  const percentage = Math.round((score / totalQuestions) * 100);

  if (!trophy) return <div className="min-h-screen bg-black" />;

  // Show personalization form first
  if (showPersonalizationForm) {
    return (
      <div className="min-h-screen bg-black text-white relative overflow-hidden">
        {/* Stadium Background */}
        <div className="fixed inset-0 z-0 opacity-20">
          <div className="absolute inset-0 bg-gradient-to-b from-green-900/30 via-black to-black" />
          <div className="absolute bottom-0 left-0 right-0 h-1/2 bg-gradient-to-t from-green-950/20 to-transparent" />
          <div className="absolute top-0 left-1/4 w-32 h-32 bg-yellow-400/10 rounded-full blur-3xl" />
          <div className="absolute top-0 right-1/4 w-32 h-32 bg-yellow-400/10 rounded-full blur-3xl" />
        </div>

        <main className="relative z-10 container mx-auto px-4 py-8 md:py-12">
          <div className="max-w-2xl mx-auto">
            <div className="text-center mb-8">
              <img 
                src="https://i.ibb.co/rGfwfvx3/TMn0-Dvx-400x400.png" 
                alt="OFC Logo" 
                className="h-20 w-20 md:h-24 md:w-24 mx-auto mb-4"
              />
              <h1 className="text-3xl md:text-5xl font-bold mb-2">
                <span className="text-[#e2ff57]">Personalize Your Trophy!</span>
              </h1>
              <p className="text-gray-400 mt-4">
                Add your name and photo to create a personalized result card
              </p>
            </div>

            <Card className="bg-gradient-to-br from-green-900/20 via-black to-black border-2 border-[#e2ff57]/40 p-8 md:p-12">
              <div className="space-y-6">
                {/* Name Input */}
                <div>
                  <Label htmlFor="userName" className="text-white text-lg mb-2 block">
                    Your Name <span className="text-red-400">*</span>
                  </Label>
                  <Input
                    id="userName"
                    type="text"
                    placeholder="Enter your name"
                    value={userName}
                    onChange={(e: React.ChangeEvent<HTMLInputElement>) => setUserName(e.target.value)}
                    className="bg-gray-900/50 border-gray-700 text-white text-lg py-6"
                  />
                </div>

                {/* Image Upload */}
                <div>
                  <Label htmlFor="userImage" className="text-white text-lg mb-2 block">
                    Your Photo (Optional)
                  </Label>
                  <div className="space-y-4">
                    <input
                      ref={fileInputRef}
                      id="userImage"
                      type="file"
                      accept="image/*"
                      onChange={handleImageUpload}
                      className="hidden"
                    />
                    <Button
                      type="button"
                      onClick={() => fileInputRef.current?.click()}
                      variant="outline"
                      className="w-full border-[#e2ff57]/50 text-white hover:bg-[#e2ff57]/10 py-6 text-lg"
                    >
                      <Upload className="mr-2" size={20} />
                      {userImage ? 'Change Photo' : 'Upload Photo'}
                    </Button>

                    {userImage && (
                      <div className="flex justify-center">
                        <div className="relative">
                          <img
                            src={userImage}
                            alt="Preview"
                            className="w-32 h-32 rounded-full object-cover border-4 border-[#e2ff57]"
                          />
                        </div>
                      </div>
                    )}
                  </div>
                </div>

                {/* Submit Button */}
                <Button
                  onClick={handleSubmitPersonalization}
                  disabled={!userName.trim()}
                  className="w-full bg-[#e2ff57] text-black hover:bg-[#e2ff57]/90 py-6 text-xl font-bold"
                >
                  Continue to Results ⚡
                </Button>

                {/* Skip Button */}
                <Button
                  onClick={() => setShowPersonalizationForm(false)}
                  variant="ghost"
                  className="w-full text-gray-400 hover:text-white py-4"
                >
                  Skip personalization
                </Button>
              </div>
            </Card>
          </div>
        </main>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-black text-white relative overflow-hidden">
      {/* Stadium Background */}
      <div className="fixed inset-0 z-0 opacity-20">
        <div className="absolute inset-0 bg-gradient-to-b from-green-900/30 via-black to-black" />
        <div className="absolute bottom-0 left-0 right-0 h-1/2 bg-gradient-to-t from-green-950/20 to-transparent" />
        {/* Spotlight effects */}
        <div className="absolute top-0 left-1/4 w-40 h-40 bg-yellow-400/20 rounded-full blur-3xl animate-pulse" />
        <div className="absolute top-0 right-1/4 w-40 h-40 bg-yellow-400/20 rounded-full blur-3xl animate-pulse" style={{ animationDelay: '1s' }} />
      </div>

      {/* Confetti Animation */}
      {showConfetti && (
        <div className="fixed inset-0 z-50 pointer-events-none">
          {[...Array(50)].map((_, i: number) => (
            <div
              key={i}
              className="absolute animate-fall"
              style={{
                left: `${Math.random() * 100}%`,
                top: '-10px',
                animationDelay: `${Math.random() * 2}s`,
                animationDuration: `${2 + Math.random() * 2}s`
              }}
            >
              {['⚽', '🏆', '⭐', '🎉'][Math.floor(Math.random() * 4)]}
            </div>
          ))}
        </div>
      )}

      <main className="relative z-10 container mx-auto px-4 py-8 md:py-12">
        <div className="max-w-3xl mx-auto">
          {/* Header */}
          <div className="text-center mb-8">
            <img 
              src="https://i.ibb.co/rGfwfvx3/TMn0-Dvx-400x400.png" 
              alt="OFC Logo" 
              className="h-20 w-20 md:h-24 md:w-24 mx-auto mb-4"
            />
            <h1 className="text-3xl md:text-5xl font-bold mb-2">
              <span className="text-[#e2ff57]">Quiz Complete!</span>
            </h1>
          </div>

          {/* Trophy Card */}
          <Card className="bg-gradient-to-br from-green-900/20 via-black to-black border-2 border-[#e2ff57]/40 p-8 md:p-12 mb-8 text-center relative overflow-hidden shadow-2xl">
            {/* Decorative elements */}
            <div className="absolute top-0 left-0 w-full h-2 bg-gradient-to-r from-transparent via-[#e2ff57]/40 to-transparent" />
            <div className="absolute bottom-0 left-0 w-full h-2 bg-gradient-to-r from-transparent via-[#e2ff57]/40 to-transparent" />
            
            {/* User Image */}
            {userImage && (
              <div className="mb-6">
                <img
                  src={userImage}
                  alt={userName}
                  className="w-24 h-24 md:w-32 md:h-32 rounded-full object-cover border-4 border-[#e2ff57] mx-auto"
                />
              </div>
            )}

            {/* User Name */}
            {userName && (
              <h3 className="text-2xl md:text-3xl font-bold mb-4 text-white">
                {userName}
              </h3>
            )}

            <TrophyDisplay trophyType={trophy.type} />

            <h2 className="text-3xl md:text-5xl font-bold mb-4" style={{ color: trophy.color }}>
              {trophy.title}
            </h2>

            <div className="text-5xl md:text-7xl font-bold mb-4">
              <span className="text-[#e2ff57]">{score}</span>
              <span className="text-gray-500"> / </span>
              <span className="text-white">{totalQuestions}</span>
            </div>

            <p className="text-xl md:text-2xl text-gray-300 mb-6">
              {trophy.message}
            </p>

            <div className="inline-block bg-gray-900/50 px-6 py-3 rounded-full border border-[#e2ff57]/30">
              <span className="text-lg font-bold text-[#e2ff57]">
                {percentage}% Correct
              </span>
            </div>

            {/* Performance message */}
            <div className="mt-8 p-4 bg-black/40 rounded-lg border border-gray-800">
              <p className="text-gray-400 text-sm md:text-base">
                {score >= 9 && "🌟 Phenomenal! You're an OFC master!"}
                {score >= 7 && score < 9 && "🎯 Great job! You really know your stuff!"}
                {score >= 5 && score < 7 && "📚 Nice work! There's more to learn!"}
                {score < 5 && "💪 Keep exploring OFC to improve your score!"}
              </p>
            </div>
          </Card>

          {/* Download Button */}
          <div className="mb-4">
            <Button
              onClick={handleDownloadImage}
              disabled={isGenerating}
              className="w-full bg-[#e2ff57] text-black hover:bg-[#e2ff57]/90 py-6 text-lg font-bold rounded-lg"
            >
              <Download className="mr-2" size={20} />
              {isGenerating ? 'Generating...' : 'Download Result'}
            </Button>
          </div>

          {/* Navigation Buttons */}
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <Button
              onClick={onRetake}
              variant="outline"
              className="border-[#e2ff57] text-[#e2ff57] hover:bg-[#e2ff57]/10 py-6 text-lg font-bold rounded-lg"
            >
              <RefreshCw className="mr-2" size={20} />
              Retake Quiz
            </Button>

            <Button
              onClick={onBackToHome}
              variant="outline"
              className="border-gray-700 hover:border-white hover:bg-white/10 py-6 text-lg font-bold rounded-lg"
            >
              <Home className="mr-2" size={20} />
              Back to Home
            </Button>
          </div>

          {/* Stats Card */}
          <Card className="mt-8 bg-black/60 border-gray-800 p-6">
            <h3 className="text-xl font-bold mb-4 text-center text-[#e2ff57]">
              Your Performance Breakdown
            </h3>
            <div className="grid grid-cols-3 gap-4 text-center">
              <div>
                <div className="text-3xl font-bold text-green-400">{score}</div>
                <div className="text-sm text-gray-400">Correct</div>
              </div>
              <div>
                <div className="text-3xl font-bold text-red-400">{totalQuestions - score}</div>
                <div className="text-sm text-gray-400">Incorrect</div>
              </div>
              <div>
                <div className="text-3xl font-bold text-[#e2ff57]">{percentage}%</div>
                <div className="text-sm text-gray-400">Accuracy</div>
              </div>
            </div>
          </Card>
        </div>
      </main>

      {/* Hidden canvas for image generation */}
      <canvas ref={canvasRef} style={{ display: 'none' }} />

      <style jsx>{`
        @keyframes fall {
          0% {
            transform: translateY(0) rotate(0deg);
            opacity: 1;
          }
          100% {
            transform: translateY(100vh) rotate(360deg);
            opacity: 0;
          }
        }
        .animate-fall {
          animation: fall linear forwards;
        }
      `}</style>
    </div>
  );
}
'use client';

import React from 'react';

interface TrophyDisplayProps {
  trophyType: 'gold' | 'silver' | 'bronze' | 'tryagain';
}

export default function TrophyDisplay({ trophyType }: TrophyDisplayProps): JSX.Element {
  const getTrophyEmoji = (): string => {
    switch (trophyType) {
      case 'gold':
        return '🏆';
      case 'silver':
        return '🥈';
      case 'bronze':
        return '🥉';
      case 'tryagain':
        return '💪';
      default:
        return '🏆';
    }
  };

  const getTrophyColor = (): string => {
    switch (trophyType) {
      case 'gold':
        return '#FFD700';
      case 'silver':
        return '#C0C0C0';
      case 'bronze':
        return '#CD7F32';
      case 'tryagain':
        return '#666666';
      default:
        return '#FFD700';
    }
  };

  return (
    <div className="relative mb-8">
      {/* Glow effect */}
      <div 
        className="absolute inset-0 blur-3xl opacity-30 animate-pulse"
        style={{ backgroundColor: getTrophyColor() }}
      />
      
      {/* Trophy container */}
      <div className="relative z-10">
        {/* Football field lines decoration */}
        <div className="absolute inset-0 flex items-center justify-center opacity-10">
          <svg width="200" height="200" viewBox="0 0 200 200" className="animate-spin-slow">
            <circle cx="100" cy="100" r="80" stroke="#e2ff57" strokeWidth="2" fill="none" />
            <circle cx="100" cy="100" r="60" stroke="#e2ff57" strokeWidth="2" fill="none" />
            <circle cx="100" cy="100" r="40" stroke="#e2ff57" strokeWidth="2" fill="none" />
            <line x1="100" y1="20" x2="100" y2="180" stroke="#e2ff57" strokeWidth="2" />
            <line x1="20" y1="100" x2="180" y2="100" stroke="#e2ff57" strokeWidth="2" />
          </svg>
        </div>

        {/* Main trophy */}
        <div className="relative text-center">
          <div 
            className="inline-block p-8 rounded-full border-4 animate-bounce-slow"
            style={{ 
              borderColor: getTrophyColor(),
              boxShadow: `0 0 40px ${getTrophyColor()}40`
            }}
          >
            <div className="text-9xl animate-wiggle">
              {getTrophyEmoji()}
            </div>
          </div>
        </div>

        {/* Sparkle effects */}
        {trophyType !== 'tryagain' && (
          <>
            <div className="absolute top-0 left-1/4 text-3xl animate-ping-slow">⭐</div>
            <div className="absolute top-1/4 right-1/4 text-2xl animate-ping-slow" style={{ animationDelay: '0.5s' }}>✨</div>
            <div className="absolute bottom-1/4 left-1/3 text-2xl animate-ping-slow" style={{ animationDelay: '1s' }}>💫</div>
          </>
        )}

        {/* Football icons */}
        <div className="absolute top-1/2 left-0 text-2xl animate-float">⚽</div>
        <div className="absolute top-1/2 right-0 text-2xl animate-float" style={{ animationDelay: '1s' }}>⚽</div>
      </div>

      <style jsx>{`
        @keyframes spin-slow {
          from {
            transform: rotate(0deg);
          }
          to {
            transform: rotate(360deg);
          }
        }
        @keyframes bounce-slow {
          0%, 100% {
            transform: translateY(0);
          }
          50% {
            transform: translateY(-20px);
          }
        }
        @keyframes wiggle {
          0%, 100% {
            transform: rotate(-5deg);
          }
          50% {
            transform: rotate(5deg);
          }
        }
        @keyframes ping-slow {
          0% {
            transform: scale(0.8);
            opacity: 0;
          }
          50% {
            transform: scale(1.2);
            opacity: 1;
          }
          100% {
            transform: scale(0.8);
            opacity: 0;
          }
        }
        @keyframes float {
          0%, 100% {
            transform: translateY(0px);
          }
          50% {
            transform: translateY(-15px);
          }
        }
        .animate-spin-slow {
          animation: spin-slow 20s linear infinite;
        }
        .animate-bounce-slow {
          animation: bounce-slow 2s ease-in-out infinite;
        }
        .animate-wiggle {
          animation: wiggle 1s ease-in-out infinite;
        }
        .animate-ping-slow {
          animation: ping-slow 3s cubic-bezier(0, 0, 0.2, 1) infinite;
        }
        .animate-float {
          animation: float 3s ease-in-out infinite;
        }
      `}</style>
    </div>
  );
}
import type { Question } from '@/types/quiz';

export const quizData: Question[] = [
  // EASY QUESTIONS (1-6)
  {
    id: 1,
    question: "What does OFC stand for?",
    options: [
      "Online Football Community",
      "OneFootball Club",
      "Official Football Channel",
      "Open Football Collective"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'basics'
  },
  {
    id: 2,
    question: "What is the portable fan passport called in the OneFootball Club ecosystem?",
    options: [
      ".football ID",
      "Fan Pass",
      "OFC Passport",
      "Football Token"
    ],
    correctAnswer: 0,
    difficulty: 'easy',
    category: 'core-features'
  },
  {
    id: 3,
    question: "What is the name of OneFootball Club's digital collectible avatar IP?",
    options: [
      "Football Stars",
      "OneFootball Heads",
      "Soccer Legends",
      "Fan Avatars"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'collectibles'
  },
  {
    id: 4,
    question: "What is the utility token used in the OneFootball Club ecosystem?",
    options: [
      "$FOOT",
      "$BALL",
      "$OFC",
      "$CLUB"
    ],
    correctAnswer: 2,
    difficulty: 'easy',
    category: 'economics'
  },
  {
    id: 5,
    question: "What is OneFootball Club's core mission?",
    options: [
      "To create football games",
      "To recognize, reward, and empower engaged football fans worldwide",
      "To organize football tournaments",
      "To sell football merchandise"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'vision'
  },
  {
    id: 6,
    question: "How many questions are in a full OFC Quiz?",
    options: [
      "5 questions",
      "10 questions",
      "15 questions",
      "20 questions"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'quiz'
  },

  // MEDIUM QUESTIONS (7-12)
  {
    id: 7,
    question: "What type of economy does OneFootball Club create for fans?",
    options: [
      "Play-to-earn economy",
      "Watch-to-earn economy",
      "Engage-to-own economy",
      "Share-to-win economy"
    ],
    correctAnswer: 2,
    difficulty: 'medium',
    category: 'economics'
  },
  {
    id: 8,
    question: "What does the .football ID store and track?",
    options: [
      "Only match predictions",
      "Achievements, reputation, and personalized experiences",
      "Just collectible ownership",
      "Only poll votes"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'core-features'
  },
  {
    id: 9,
    question: "Which of the following is NOT a way fans earn rewards in OneFootball Club?",
    options: [
      "Match predictions",
      "Community challenges",
      "Watching advertisements",
      "Streak-based actions"
    ],
    correctAnswer: 2,
    difficulty: 'medium',
    category: 'gamification'
  },
  {
    id: 10,
    question: "What unlocks access to exclusive drops and higher-tier rewards?",
    options: [
      "Paying a subscription fee",
      "Stronger fan reputation scores",
      "Inviting friends",
      "Following on social media"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'gamification'
  },
  {
    id: 11,
    question: "The .football ID is built on which technology?",
    options: [
      "Cloud storage",
      "Traditional databases",
      "Blockchain (on-chain)",
      "Mobile apps only"
    ],
    correctAnswer: 2,
    difficulty: 'medium',
    category: 'technology'
  },
  {
    id: 12,
    question: "What does OneFootball Club use to incentivize fan knowledge and loyalty?",
    options: [
      "Physical merchandise only",
      "$OFC rewards and tokens",
      "Discount coupons",
      "Free tickets"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'economics'
  },

  // HARD QUESTIONS (13-22)
  {
    id: 13,
    question: "OneFootball Club represents a fusion of which three key elements?",
    options: [
      "Gaming, shopping, and streaming",
      "Fandom, finance, and ownership",
      "Betting, trading, and collecting",
      "Social media, advertising, and content"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 14,
    question: "What is the primary innovation that OneFootball Club brings to football fandom?",
    options: [
      "Free streaming of matches",
      "Fans can participate in the value creation of their own engagement",
      "Cheaper merchandise",
      "Better match statistics"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 15,
    question: "How does OneFootball Club transform fan engagement?",
    options: [
      "From active ownership to passive consumption",
      "From passive consumption to active ownership",
      "From paid access to free access",
      "From online to offline only"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 16,
    question: "According to OneFootball Club's gamification system, what does the platform track to create unique fan profiles?",
    options: [
      "Only social media followers",
      "Fan activity that evolves over time",
      "Credit card purchases",
      "Match attendance only"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'gamification'
  },
  {
    id: 17,
    question: "What is the main purpose of the Fan Dashboard in the OneFootball Club ecosystem?",
    options: [
      "To watch live matches",
      "To track and manage fan engagement, XP, badges, and rewards",
      "To buy football tickets",
      "To chat with other fans"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'core-features'
  },
  {
    id: 18,
    question: "What makes the .football ID a 'persistent digital legacy'?",
    options: [
      "It expires after one year",
      "It captures a fan's journey across the ecosystem over time",
      "It only works in one app",
      "It's stored on your phone only"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'core-features'
  },
  {
    id: 19,
    question: "In the OneFootball Club model, speculation becomes ______, engagement becomes ______, and fan identity becomes ______.",
    options: [
      "Random, boring, worthless",
      "Skill-based, ownable, capital",
      "Easy, simple, free",
      "Complex, difficult, expensive"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 20,
    question: "What is the overarching vision of OneFootball Club?",
    options: [
      "To create the best football streaming platform",
      "To empower the world's largest community of football fans by breaking down barriers and creating value",
      "To organize international tournaments",
      "To replace traditional football clubs"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 21,
    question: "What does OneFootball Club use leaderboards for?",
    options: [
      "Only to rank teams",
      "To let fans compete globally for rewards, achievements, and exclusive benefits",
      "To display match scores",
      "To show player statistics"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'gamification'
  },
  {
    id: 22,
    question: "The .football Vault is designed to store:",
    options: [
      "Match videos only",
      "Digital collectibles and fan assets",
      "Personal photos",
      "Team merchandise"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'core-features'
  },

  // ADDITIONAL EASY QUESTIONS (23-32)
  {
    id: 23,
    question: "What blockchain network does OneFootball Club primarily use?",
    options: [
      "Ethereum",
      "Base (Layer 2)",
      "Bitcoin",
      "Solana"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'technology'
  },
  {
    id: 24,
    question: "What can fans earn by completing challenges on OneFootball Club?",
    options: [
      "Only badges",
      "XP and rewards",
      "Physical trophies",
      "Nothing"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'gamification'
  },
  {
    id: 25,
    question: "What does XP stand for in the OneFootball Club ecosystem?",
    options: [
      "Extra Points",
      "Experience Points",
      "Exclusive Privileges",
      "Expert Player"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'gamification'
  },
  {
    id: 26,
    question: "Can fans trade their OneFootball Heads avatars?",
    options: [
      "No, they are non-transferable",
      "Yes, they are digital collectibles that can be traded",
      "Only with permission",
      "Only after one year"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'collectibles'
  },
  {
    id: 27,
    question: "What is the primary goal of the Fan Dashboard?",
    options: [
      "To watch live matches",
      "To centralize and visualize fan engagement and rewards",
      "To buy merchandise",
      "To chat with players"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'core-features'
  },
  {
    id: 28,
    question: "What industry challenge does OneFootball Club address?",
    options: [
      "Lack of football content",
      "Fans being excluded from value creation in football",
      "Too many football apps",
      "High ticket prices"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'vision'
  },
  {
    id: 29,
    question: "Are OneFootball Credits the same as $OFC tokens?",
    options: [
      "Yes, they are identical",
      "No, Credits are off-chain while $OFC is on-chain",
      "They don't exist",
      "Only Credits exist"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'economics'
  },
  {
    id: 30,
    question: "What can fans do with their .football ID?",
    options: [
      "Only login to one app",
      "Use it across multiple platforms and experiences",
      "Nothing special",
      "Only view their profile"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'core-features'
  },
  {
    id: 31,
    question: "What type of predictions can fans make on OneFootball Club?",
    options: [
      "Weather predictions",
      "Gamified match predictions with rewards",
      "Stock market predictions",
      "No predictions allowed"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'gamification'
  },
  {
    id: 32,
    question: "What visual style do OneFootball Heads avatars have?",
    options: [
      "Realistic photographs",
      "Stylized, collectible digital art",
      "8-bit pixel art",
      "Hand-drawn sketches"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'collectibles'
  },

  // ADDITIONAL MEDIUM QUESTIONS (33-44)
  {
    id: 33,
    question: "How does the tier system work in OneFootball Club?",
    options: [
      "Random assignment",
      "Based on fan activity, XP accumulation, and engagement levels",
      "By paying subscription fees",
      "First come, first served"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'gamification'
  },
  {
    id: 34,
    question: "What happens when fans complete streak-based challenges?",
    options: [
      "Nothing happens",
      "They earn bonus rewards and unlock exclusive content",
      "They lose points",
      "They get banned"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'gamification'
  },
  {
    id: 35,
    question: "The .football ID creates which type of fan profile?",
    options: [
      "Static and unchanging",
      "Dynamic and evolving based on engagement",
      "Temporary and disposable",
      "Anonymous only"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'core-features'
  },
  {
    id: 36,
    question: "What makes OneFootball Club different from traditional fan apps?",
    options: [
      "Better graphics",
      "Fans own their engagement and earn from their loyalty",
      "More advertisements",
      "Cheaper subscription"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'vision'
  },
  {
    id: 37,
    question: "Badges in OneFootball Club represent:",
    options: [
      "Random decorations",
      "Achievements and milestones earned through engagement",
      "Purchased items only",
      "Admin privileges"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'gamification'
  },
  {
    id: 38,
    question: "What is the relationship between XP and leveling up?",
    options: [
      "No relationship",
      "Accumulating XP allows fans to level up and unlock new tiers",
      "XP decreases your level",
      "Only paying users can level up"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'gamification'
  },
  {
    id: 39,
    question: "How portable is the .football ID?",
    options: [
      "Not portable at all",
      "Fully portable across the entire OneFootball ecosystem",
      "Only works on mobile",
      "Requires re-registration everywhere"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'core-features'
  },
  {
    id: 40,
    question: "What type of economy does $OFC token enable?",
    options: [
      "Closed, centralized economy",
      "Open, community-driven token economy",
      "Government-controlled economy",
      "No economy at all"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'economics'
  },
  {
    id: 41,
    question: "Fan reputation in OneFootball Club is determined by:",
    options: [
      "Social media followers only",
      "Consistent engagement, predictions, challenges, and community participation",
      "Random selection",
      "Account age only"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'gamification'
  },
  {
    id: 42,
    question: "What does 'engage-to-own' mean in the OneFootball Club context?",
    options: [
      "Fans must buy everything",
      "Fan engagement translates into ownership of value and digital assets",
      "Fans can only rent content",
      "Engagement has no value"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'vision'
  },
  {
    id: 43,
    question: "OneFootball Credits can be converted to:",
    options: [
      "Nothing",
      "$OFC tokens when available",
      "Physical cash only",
      "Gift cards only"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'economics'
  },
  {
    id: 44,
    question: "The leaderboard system allows fans to:",
    options: [
      "Only view their own rank",
      "Compete globally, regionally, and among friends for rewards",
      "Play video games",
      "Watch highlights"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'gamification'
  },

  // ADDITIONAL HARD QUESTIONS (45-62)
  {
    id: 45,
    question: "OneFootball Club's vision is to transform football fandom from extraction to:",
    options: [
      "Subscription",
      "Redistribution and empowerment",
      "Advertising",
      "Monetization"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 46,
    question: "What blockchain standard is the .football ID built on?",
    options: [
      "Traditional web2 databases",
      "On-chain identity standard with portability",
      "Email verification only",
      "SMS authentication"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'technology'
  },
  {
    id: 47,
    question: "The Fan Dashboard serves as:",
    options: [
      "Just a profile page",
      "A comprehensive hub for tracking engagement, assets, achievements, and rewards across the ecosystem",
      "An advertising platform",
      "A payment gateway"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'core-features'
  },
  {
    id: 48,
    question: "OneFootball Club redefines value in football by:",
    options: [
      "Lowering ticket prices",
      "Allowing fans to capture value from their own engagement and loyalty",
      "Creating more advertisements",
      "Increasing merchandise costs"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 49,
    question: "What is the significance of the gamification shift in OneFootball Club?",
    options: [
      "It's just for fun",
      "It transforms passive fans into active participants who earn rewards for their engagement",
      "It makes the app harder to use",
      "It removes all content"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'gamification'
  },
  {
    id: 50,
    question: "The .football Vault enables:",
    options: [
      "Only viewing digital items",
      "True ownership and secure storage of digital collectibles with full control",
      "Renting digital assets",
      "Temporary access only"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'core-features'
  },
  {
    id: 51,
    question: "OneFootball Heads are unique because they:",
    options: [
      "Look the same for everyone",
      "Represent fan identity and can evolve with achievements and milestones",
      "Are temporary profile pictures",
      "Cannot be customized"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'collectibles'
  },
  {
    id: 52,
    question: "The redistribution of value in OneFootball Club means:",
    options: [
      "Lower prices for everyone",
      "Value flows back to fans rather than being captured only by platforms",
      "Higher fees for users",
      "No change in value distribution"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'economics'
  },
  {
    id: 53,
    question: "What problem does OneFootball Club solve in the current football industry?",
    options: [
      "Lack of football matches",
      "Fans are treated as consumers rather than stakeholders in the ecosystem",
      "Too many football clubs",
      "Expensive streaming services only"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 54,
    question: "The tier system in OneFootball Club unlocks:",
    options: [
      "Nothing special",
      "Progressively better rewards, exclusive drops, and premium experiences",
      "Only cosmetic changes",
      "Paid subscriptions"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'gamification'
  },
  {
    id: 55,
    question: "Fan identity as capital means:",
    options: [
      "Fans must invest money",
      "A fan's engagement history and reputation have tangible value and unlock opportunities",
      "Identity has no value",
      "Only verified celebrities benefit"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 56,
    question: "OneFootball Club's approach to fan engagement is:",
    options: [
      "Traditional advertising model",
      "Community-first, rewarding engagement rather than extracting value",
      "Pay-per-view only",
      "Subscription-based without rewards"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 57,
    question: "The on-chain nature of .football ID provides:",
    options: [
      "No benefits",
      "Transparency, portability, and true ownership of fan identity",
      "Slower performance",
      "Higher costs only"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'technology'
  },
  {
    id: 58,
    question: "Gamified predictions differ from traditional betting by:",
    options: [
      "They are identical",
      "Focusing on skill, knowledge, and community engagement rather than pure gambling",
      "Having worse odds",
      "Being illegal"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'gamification'
  },
  {
    id: 59,
    question: "What makes OneFootball Club a Web3 platform?",
    options: [
      "It uses websites",
      "Built on blockchain with tokenization, digital ownership, and decentralized value distribution",
      "It has a mobile app",
      "It uses social media"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'technology'
  },
  {
    id: 60,
    question: "The ultimate goal of OneFootball Club is to:",
    options: [
      "Compete with streaming services",
      "Build the world's largest and most engaged football community where fans are stakeholders",
      "Sell more merchandise",
      "Replace football clubs"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 61,
    question: "Community challenges in OneFootball Club create:",
    options: [
      "Individual competition only",
      "Collaborative experiences where fans work together for shared rewards",
      "No interaction",
      "Only leaderboard rankings"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'gamification'
  },
  {
    id: 62,
    question: "The intersection of fandom, finance, and ownership in OneFootball Club results in:",
    options: [
      "Confusion for users",
      "A revolutionary model where passionate fans become empowered stakeholders",
      "Traditional fan experiences",
      "Higher costs for fans"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },

  // MORE EASY QUESTIONS (63-74)
  {
    id: 63,
    question: "What type of rewards can fans earn in OneFootball Club?",
    options: [
      "Only virtual trophies",
      "XP, badges, $OFC tokens, and exclusive collectibles",
      "Physical merchandise only",
      "Nothing"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'gamification'
  },
  {
    id: 64,
    question: "Can the .football ID be used across multiple platforms?",
    options: [
      "No, it's locked to one app",
      "Yes, it's portable across the entire OneFootball ecosystem",
      "Only on desktop",
      "Only on mobile"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'core-features'
  },
  {
    id: 65,
    question: "What makes OneFootball Club a community-first platform?",
    options: [
      "It has many users",
      "Fans are rewarded for engagement and have ownership in the ecosystem",
      "It's free to use",
      "It has chat features"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'vision'
  },
  {
    id: 66,
    question: "OneFootball Heads avatars are stored:",
    options: [
      "On your phone only",
      "In the .football Vault on-chain",
      "On social media",
      "In email attachments"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'collectibles'
  },
  {
    id: 67,
    question: "What does completing daily challenges reward you with?",
    options: [
      "Nothing",
      "XP, badges, and potential token rewards",
      "Only a thank you message",
      "Ads removal"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'gamification'
  },
  {
    id: 68,
    question: "Is OneFootball Club available globally?",
    options: [
      "No, only in Europe",
      "Yes, it aims to serve football fans worldwide",
      "Only in Asia",
      "Only in North America"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'basics'
  },
  {
    id: 69,
    question: "What is the main focus of OneFootball Club's gamification?",
    options: [
      "Playing video games",
      "Rewarding real fan engagement and loyalty",
      "Watching movies",
      "Shopping"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'gamification'
  },
  {
    id: 70,
    question: "How does OneFootball Club treat its fans?",
    options: [
      "As passive consumers",
      "As active stakeholders and value creators",
      "As advertisers",
      "As outsiders"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'vision'
  },
  {
    id: 71,
    question: "What technology powers the .football ID?",
    options: [
      "Email servers",
      "Blockchain technology",
      "Paper records",
      "USB drives"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'technology'
  },
  {
    id: 72,
    question: "Can fans compete with friends on OneFootball Club?",
    options: [
      "No, only global competition",
      "Yes, through leaderboards and challenges",
      "Only in person",
      "Competition is not allowed"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'gamification'
  },
  {
    id: 73,
    question: "What is the purpose of the .football Vault?",
    options: [
      "To store passwords",
      "To securely store and manage digital collectibles and fan assets",
      "To save articles",
      "To backup photos"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'core-features'
  },
  {
    id: 74,
    question: "Does OneFootball Club offer personalized experiences?",
    options: [
      "No, everyone sees the same content",
      "Yes, based on your .football ID and engagement history",
      "Only for premium users",
      "Personalization is not available"
    ],
    correctAnswer: 1,
    difficulty: 'easy',
    category: 'core-features'
  },

  // MORE MEDIUM QUESTIONS (75-87)
  {
    id: 75,
    question: "How does the streak system encourage fan engagement?",
    options: [
      "It doesn't",
      "By rewarding consistent daily activity with bonus multipliers and exclusive rewards",
      "By punishing inactivity",
      "By sending emails"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'gamification'
  },
  {
    id: 76,
    question: "What role do OneFootball Heads play in the ecosystem?",
    options: [
      "Just profile pictures",
      "Digital avatars that represent fan identity, unlock perks, and evolve with achievements",
      "Game characters only",
      "Advertisement placeholders"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'collectibles'
  },
  {
    id: 77,
    question: "How does OneFootball Club ensure fair value distribution?",
    options: [
      "It doesn't matter",
      "Through transparent on-chain systems and community-driven tokenomics",
      "By manual approval",
      "Random selection"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'economics'
  },
  {
    id: 78,
    question: "What happens when you level up in OneFootball Club?",
    options: [
      "Nothing changes",
      "You unlock new tiers, better rewards, and exclusive experiences",
      "You lose progress",
      "You have to pay fees"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'gamification'
  },
  {
    id: 79,
    question: "The Fan Dashboard allows you to:",
    options: [
      "Only view news",
      "Track all your engagement, XP, badges, collectibles, and rewards in one place",
      "Just login",
      "Watch live matches only"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'core-features'
  },
  {
    id: 80,
    question: "Why is Base chosen as the primary blockchain for OneFootball Club?",
    options: [
      "Random choice",
      "Low fees, fast transactions, and Ethereum security with Layer 2 scalability",
      "Only option available",
      "Most expensive option"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'technology'
  },
  {
    id: 81,
    question: "How do badges contribute to fan progression?",
    options: [
      "They don't",
      "They showcase achievements, boost reputation, and unlock tier upgrades",
      "Only for decoration",
      "They cost money"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'gamification'
  },
  {
    id: 82,
    question: "What distinguishes $OFC from traditional fan tokens?",
    options: [
      "Nothing special",
      "It's earned through engagement, not just purchased, and powers a full ecosystem",
      "It's more expensive",
      "It's centralized"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'economics'
  },
  {
    id: 83,
    question: "The .football ID creates value for fans by:",
    options: [
      "Displaying their name",
      "Building a portable reputation and unlocking progressive benefits across platforms",
      "Just storing login details",
      "Showing profile pictures"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'core-features'
  },
  {
    id: 84,
    question: "How do global leaderboards enhance competition?",
    options: [
      "They don't",
      "By letting fans compete worldwide for top rankings, rewards, and recognition",
      "By hiding scores",
      "By limiting participation"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'gamification'
  },
  {
    id: 85,
    question: "What makes OneFootball Club predictions skill-based?",
    options: [
      "Pure luck",
      "They reward football knowledge, analysis, and consistent accuracy over time",
      "Random outcomes",
      "No skill involved"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'gamification'
  },
  {
    id: 86,
    question: "The on-chain nature of fan assets ensures:",
    options: [
      "Nothing special",
      "True ownership, transparency, and portability across platforms",
      "Higher costs",
      "Slower access"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'technology'
  },
  {
    id: 87,
    question: "How does OneFootball Club empower football fans?",
    options: [
      "By giving them news",
      "By letting them own their engagement, earn from loyalty, and participate in value creation",
      "By showing ads",
      "By selling merchandise"
    ],
    correctAnswer: 1,
    difficulty: 'medium',
    category: 'vision'
  },

  // MORE HARD QUESTIONS (88-100)
  {
    id: 88,
    question: "The concept of 'fan identity as capital' in OneFootball Club means:",
    options: [
      "Fans must invest capital",
      "A fan's accumulated reputation, achievements, and engagement history become valuable assets",
      "Identity has no economic value",
      "Only wealthy fans benefit"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 89,
    question: "How does OneFootball Club address the traditional football industry's value extraction problem?",
    options: [
      "It doesn't address it",
      "By creating mechanisms where fans capture value from their own engagement instead of platforms extracting it",
      "By charging higher fees",
      "By limiting access"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 90,
    question: "The tier system progression is designed to:",
    options: [
      "Create barriers",
      "Reward sustained engagement with escalating benefits, creating long-term fan investment",
      "Limit participation",
      "Generate subscription fees"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'gamification'
  },
  {
    id: 91,
    question: "What strategic advantage does Base (Layer 2) provide OneFootball Club?",
    options: [
      "No advantages",
      "Combines Ethereum security with high throughput, low costs, and mainstream accessibility",
      "Higher transaction fees",
      "Slower confirmation times"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'technology'
  },
  {
    id: 92,
    question: "The Fan Dashboard's comprehensive tracking creates value by:",
    options: [
      "Just displaying information",
      "Providing transparent proof of engagement that translates into reputation, rewards, and opportunities",
      "Collecting data for ads",
      "Showing random statistics"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'core-features'
  },
  {
    id: 93,
    question: "OneFootball Club's approach to digital collectibles differs from typical NFTs by:",
    options: [
      "Being identical",
      "Integrating utility, identity, and progressive unlocks tied to real fan engagement",
      "Being more expensive",
      "Having no utility"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'collectibles'
  },
  {
    id: 94,
    question: "The redistribution mechanism in OneFootball Club works by:",
    options: [
      "Giving everyone equal amounts",
      "Allocating value based on contribution, engagement, and community participation",
      "Random distribution",
      "Only benefiting early adopters"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'economics'
  },
  {
    id: 95,
    question: "How does the XP system create sustainable engagement?",
    options: [
      "By forcing participation",
      "By rewarding diverse activities, creating progression paths, and linking to tangible benefits",
      "By punishing inactivity",
      "By limiting earning potential"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'gamification'
  },
  {
    id: 96,
    question: "The .football ID's portability across platforms enables:",
    options: [
      "Nothing special",
      "A unified fan experience where reputation and assets travel seamlessly across the ecosystem",
      "Platform lock-in",
      "Data silos"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'core-features'
  },
  {
    id: 97,
    question: "OneFootball Club's vision challenges the traditional sports industry by:",
    options: [
      "Competing on content only",
      "Fundamentally restructuring value flows to prioritize fan ownership and participation",
      "Copying existing models",
      "Focusing on advertising"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  },
  {
    id: 98,
    question: "The relationship between OneFootball Credits and $OFC tokens represents:",
    options: [
      "No connection",
      "A bridge between off-chain engagement and on-chain value, enabling seamless transition",
      "Two competing systems",
      "Redundant features"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'economics'
  },
  {
    id: 99,
    question: "How do community challenges differ from individual challenges in creating value?",
    options: [
      "They don't differ",
      "They foster collective achievement, strengthen community bonds, and create shared rewards",
      "They're harder",
      "They pay less"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'gamification'
  },
  {
    id: 100,
    question: "OneFootball Club's ultimate transformation of football fandom achieves:",
    options: [
      "Better content delivery",
      "A paradigm shift where fans evolve from passive consumers to empowered stakeholders with ownership",
      "More social features",
      "Cheaper access"
    ],
    correctAnswer: 1,
    difficulty: 'hard',
    category: 'vision'
  }
];
export interface Question {
  id: number;
  question: string;
  options: string[];
  correctAnswer: number;
  difficulty: 'easy' | 'medium' | 'hard';
  category: string;
}

export interface QuizResult {
  score: number;
  totalQuestions: number;
  trophyType: 'gold' | 'silver' | 'bronze' | 'tryagain';
  message: string;
}

export interface TrophyData {
  type: 'gold' | 'silver' | 'bronze' | 'tryagain';
  title: string;
  message: string;
  emoji: string;
  color: string;
}
{
  "name": "mini-app",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@farcaster/miniapp-sdk": "^0.2.1",
    "@farcaster/quick-auth": "^0.0.8",
    "@hookform/resolvers": "^5.0.1",
    "@prisma/client": "^5.11.0",
    "@radix-ui/react-accordion": "^1.2.10",
    "@radix-ui/react-alert-dialog": "^1.1.13",
    "@radix-ui/react-aspect-ratio": "^1.1.6",
    "@radix-ui/react-avatar": "^1.1.9",
    "@radix-ui/react-checkbox": "^1.3.1",
    "@radix-ui/react-collapsible": "^1.1.10",
    "@radix-ui/react-context-menu": "^2.2.14",
    "@radix-ui/react-dialog": "^1.1.13",
    "@radix-ui/react-dropdown-menu": "^2.1.14",
    "@radix-ui/react-hover-card": "^1.1.13",
    "@radix-ui/react-label": "^2.1.6",
    "@radix-ui/react-menubar": "^1.1.14",
    "@radix-ui/react-navigation-menu": "^1.2.12",
    "@radix-ui/react-popover": "^1.1.13",
    "@radix-ui/react-progress": "^1.1.6",
    "@radix-ui/react-radio-group": "^1.3.6",
    "@radix-ui/react-scroll-area": "^1.2.8",
    "@radix-ui/react-select": "^2.2.4",
    "@radix-ui/react-separator": "^1.1.6",
    "@radix-ui/react-slider": "^1.3.4",
    "@radix-ui/react-slot": "^1.2.2",
    "@radix-ui/react-switch": "^1.2.4",
    "@radix-ui/react-tabs": "^1.1.11",
    "@radix-ui/react-toggle": "^1.1.8",
    "@radix-ui/react-toggle-group": "^1.1.9",
    "@radix-ui/react-tooltip": "^1.2.6",
    "@types/request": "^2.48.12",
    "class-variance-authority": "^0.7.1",
    "clsx": "^2.1.1",
    "cmdk": "^1.1.1",
    "date-fns": "^3.6.0",
    "embla-carousel-react": "^8.6.0",
    "framer-motion": "^12.12.1",
    "input-otp": "^1.4.2",
    "lucide-react": "^0.511.0",
    "next": "15.3.4",
    "next-themes": "^0.4.6",
    "posthog-js": "^1.242.2",
    "react": "19.1.0",
    "react-day-picker": "^8.10.1",
    "react-dom": "19.1.0",
    "react-hook-form": "^7.56.3",
    "react-resizable-panels": "^3.0.2",
    "recharts": "^2.15.3",
    "request": "^2.88.2",
    "sonner": "^2.0.3",
    "tailwind-merge": "^3.3.0",
    "tailwindcss-animate": "^1.0.7",
    "tough-cookie": "^5.1.2",
    "vaul": "^1.1.2",
    "winston": "^3.17.0",
    "zod": "^3.24.4"
  },
  "devDependencies": {
    "@anton-seriesfi/ts-typie": "^3.0.2",
    "@types/date-fns": "^2.6.3",
    "@types/node": "20.17.47",
    "@types/react": "19.1.8",
    "@types/react-dom": "19.1.6",
    "autoprefixer": "^10.4.21",
    "depcheck": "^1.4.7",
    "postcss": "^8",
    "prisma": "^5.11.0",
    "tailwindcss": "^3.4.1",
    "typescript": "5.8.3"
  },
  "overrides": {
    "@types/react": "19.1.8",
    "@types/react-dom": "19.1.6"
  }
}
{
  "miniapp": {
    "version": "1",
    "name": "OFC Quiz Championship",
    "iconUrl": "https://usdozf7pplhxfvrl.public.blob.vercel-storage.com/thumbnail_e4bff3c9-a02c-41e7-aea5-7f6e1a823ea0-FDEiakQfcRN74qt3F5lFUcCEzpYUuD",
    "homeUrl": "https://farmer-station-266.app.ohara.ai",
    "imageUrl": "https://usdozf7pplhxfvrl.public.blob.vercel-storage.com/thumbnail_e4bff3c9-a02c-41e7-aea5-7f6e1a823ea0-FDEiakQfcRN74qt3F5lFUcCEzpYUuD",
    "buttonTitle": "Open with Ohara",
    "splashImageUrl": "https://usdozf7pplhxfvrl.public.blob.vercel-storage.com/farcaster/splash_images/splash_image1.svg",
    "splashBackgroundColor": "#ffffff",
    "tags": [
      "ohara",
      "mini-app",
      "base",
      "farcaster"
    ],
    "primaryCategory": "entertainment",
    "ogTitle": "OFC Quiz Championship",
    "ogImageUrl": "https://usdozf7pplhxfvrl.public.blob.vercel-storage.com/thumbnail_e4bff3c9-a02c-41e7-aea5-7f6e1a823ea0-FDEiakQfcRN74qt3F5lFUcCEzpYUuD"
  },
  "baseBuilder": {
    "allowedAddresses": [
      ""
    ]
  }
}

